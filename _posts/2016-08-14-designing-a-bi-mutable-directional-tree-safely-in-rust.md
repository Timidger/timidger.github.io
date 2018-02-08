---
layout: post
title: Designing a mutable bi-directional tree safely in Rust
date: 2016-08-14
categories: blog
---

While designing [Way Cooler](https://github.com/Immington-Industries/way-cooler), it was decided early on that we wanted multiple different ways for the user to tile their windows. When you look around at all the [different](http://dwm.suckless.org/) [tiling](https://awesome.naquadah.org/) [window](http://i3wm.org/) [managers](http://xmonad.org/) for X, it becomes apparent that there is no one-size-fits-all method. One of the primary goals of Way Cooler is to be as customizable as possible, so we want the user to choose the method that works best for them.

Since the window manager I'm most familiar and comfortable with is i3, I decided to implement that first. i3 manages the windows on the screen by constructing a [virtual tree](http://i3wm.org/docs/userguide.html#_tree) that the user can manipulate by adding containers and switching how each container lays out its children. Unlike Awesome's static layout templates or Xmonad's non-structured horizontal/vertical/fullscreen tiling, i3 lets you construct a layout yourself. i3's commands are so simple, therefore it will be trivial to either augment them or strip them away completely when we offer more tiling options to the user.

## Designing the data definition using smart pointers
[Rust](https://www.rust-lang.org/en-US/), our language of choice for Way Cooler, encourages a data-orientated approach to programming. Since our tree had to be flexible enough to accommodate all these different tiling styles, our data structure had to be designed to be as flexible as possible. After taking a good long look at the i3 docs, we came up with the following structure for a node in the tree:

```rust
pub struct Node {
    parent: Option<Weak<RefCell<Node>>>,
    children: Vec<Rc<RefCell<Node>>>,
    // eliding other fields
}
```
Needless to say, those fields are very complicated. What are all those wrappers doing? Lets break it down and see just what kind of a mess we've gotten ourselves into.


A parent is wrapped in an [Option](https://doc.rust-lang.org/std/option/enum.Option.html) because it's possible for the `Node` to be the root of the tree, which doesn't have a parent. We could define our tree so that the root just refers to itself, but not only can that be messy [if we change to bare references](http://stackoverflow.com/questions/32300132/why-cant-i-store-a-value-and-a-reference-to-that-value-in-the-same-struct), it also doesn't force us to check if we have exhausted our search of the tree.


Next is [`Rc`](https://doc.rust-lang.org/std/rc/struct.Rc.html) and [`Weak`](https://doc.rust-lang.org/std/rc/struct.Weak.html). These types are actually referring to the same thing: a reference counted smart pointer. We can't use bare references here because Rust won't let us modify the tree and invalidate its internal references to itself because that would be unsafe.  When you `.clone())` an `Rc`, instead of copying the underlying value it increments an internal counter and lets us happily use and destroy the `Rc` because all we have is a pointer to the data. The value behind the pointer is only dropped when the counter reaches 0, so an `Rc` will never attempt to access a dangling pointer.

When an `Rc` is downgraded to a `Weak` it decrements the reference count but keeps the pointer to the value in the `Weak`. Whenever you try to dereference the `Weak`, you have to check if it still exists since your reference isn't keeping it alive anymore. So whenever you move up the tree you have to check if your branch is still part of the tree.<sup><a name="weak-parent-back" href="#weak-parent">1</a></sup>


A [`RefCell`](https://doc.rust-lang.org/std/cell/struct.RefCell.html) is used whenever you want to be able to mutate a field of a struct but you are unable or unwilling to own or mutably borrow the whole struct. Since we are constructing a tree, most of our operations are defined recursively, which means we can't mutably borrow a container and its children at the same time. The borrow checker will infer that because we have a reference to a `Node` and a child of that `Node`, we could hypothetically delete the parent `Node` and cause our reference to become a dangling pointer. 

This is a very helpful and powerful smart pointer, but it comes with a huge cost. All of the borrow checking rules will now happen at run time, where a non-unique mutable borrow will cause a panic. This makes it very easy to write invalid code because the compiler won't tell us we are making a mistake until it blows up in our face.

Ok, that was a lot to take in, but it's not too bad when you look at it from a 10,000 foot view. To recap:
* A `Node` has a parent `Node` and an arbitrary number of children `Node.`
* The parent `Node` may or may not exist, depending on if we are at the top of the tree.
* A `Node` owns its children and needs to check if it is still part of the tree whenever it traverses up through its parent.
* Finally we use a `RefCell` to put the borrow checking at runtime because we cannot statically verify the safety of the tree.

Alright, lets see what some code using this data structure looks like. Lets look at something simple, like adding a child `Node` to an already existing `Node`

```rust
pub fn add_child(parent: Node, child: Node) {
    child.borrow_mut().set_parent(Rc::downgrade(&parent.clone()));
    parent.borrow_mut().children.push(child);
}
```

Even for such a simple example, it's obvious that this is going to get very tedious. Here's a direct example from our old code for adding new containers, which has more complicated logic. Don't worry about understanding it, just notice how many explicit drops and `borrow_mut`'s are needed.<sup><a name="smart-pointer-tree-code-back" href="#smart-pointer-tree-code">2</a></sup>


```rust
pub fn new_container(parent_: &mut Node, mut layout: Layout) -> Node {
    let mut parent = parent_.borrow_mut();
    let container = Rc::new(RefCell::new(Container {
        parent: Some(Rc::downgrade(&parent_)),
        children: vec!(),
        container_type: ContainerType::Container,
        layout: Some(layout),
    }));
    if parent.get_type() == ContainerType::Workspace {
        let mut workspace = parent;
        if let Some(layout) = workspace.get_layout() {
            container.borrow_mut().set_layout(layout);
        }
        workspace.add_child(container.clone());
    } else {
        // Need to add the "parent" as the child of the container we just made.
        drop(parent);
        let child_ = parent_.clone();
        let child = child_.borrow();
        let parent_of_child = child.get_parent().expect("child had no parent");
        let child_clone = Rc::make_mut(&mut child_.clone()).clone().into_inner();
        // Need to remove the child from their parent
        parent_of_child.borrow_mut().remove_child(&child_clone.clone());
        // Add the container we just made as a child, replacing the one we removed
        parent_of_child.borrow_mut().add_child(container.clone());
        // The new container is now a child of the old child's parent
        container.borrow_mut().add_child(child_.clone());
    }
    container
}
```

This tree, though memory safe, is very tedious to use and could fall apart at any time if we forget to drop a `RefCell` before trying to borrow it again. It also wouldn't be easy to extend to work with the other tiling methods that we want to support. What we need is a way to mutate part of the tree without the need for `RefCell`.

## Gaining flexibility with custom unsafe abstractions
Because Rust is a systems programming language it gives us the tools to make our own safe abstractions. Raw pointers (`*const` and `*mut`) are like references but they do not check the special borrow checker rules at compile time. It's up to the programmer to ensure that these rules are enforced in his code. The rules themselves are very simple, but can be tricky to get right.


Here's our tree, converted to using unsafe raw pointers:

```rust
pub struct Node {
    parent: *mut Node,
    children: Vec<Node>
}
```

This is much better looking, though now we need to ensure that we are following those rules manually.

Since our parent `Node` is now behind a `*mut`, Rust will no longer ensure that the value it points to is still valid.. This is bad but fixable. Since `Node`s own their children (they are moved into the `Vec` and are no longer behind an `Rc`) whenever we delete a `Node` all of its children are immediately dropped as well. To ensure that there is absolutely no way a child can point to a removed parent, we'll override `Drop` for `Node`:

```rust
impl Drop for Node {
    fn drop(&mut self) {
        let children: &mut Vec<Node> = &mut self.children;
        for mut child in children {
            child.parent = ptr::null_mut();
        }
    }
}
```

Now, thanks to RAII, there is no way for a child to have a dangling reference to a parent that was removed. Excellent! We solved the memory unsafety right?

##  Guaranteeing memory safety is hard
Unfortunately, no. This unsafe abstraction is very leaky, and prone to bugs even with totally "safe" code. A `Node` borrowed as mutable is no longer checked by the borrow checker, which means we can no longe ensure the following rules are upheld:
* One or more immutable references `&T`) to a resource and no mutable references.
* Exactly one mutable reference (`&mut T`) and no immutable references.

Because of the raw pointer we can't guarantee that a `*mut` pointer is uniquely referencing our parent `Node`, so our own definiton of `as_mut` to mirror `RefCell`'s requires the unsafe keyword:

```rust
/// Attempts to get the Node as a mutable reference by going through it's parent
///
/// # Unsafety
/// The borrow checker can not properly infer that you are taking a mutable reference
/// to this Node, so it is possible to invalidate any other references to this
/// Node when you use this method.
pub unsafe fn as_mut<'a>(&'a self) -> &'a mut Node {
    let maybe_parent = self.get_parent();
    if maybe_parent.is_some() {
        let parent: &'a mut Node = self.get_parent().unwrap();
        for child in parent.get_children_mut() {
            if *child == *self {
                return child;
            }
        };
    }
    panic!("Parent had no node!");
}
```

It is up to the caller to ensure that there are no other references to this `Node` or any of this `Node` children. If there are, then those references might find themselves refering to inconsistent data or even a dangling pointer!<sup><a href="#as_mut_gist" name="as_mut_gist_back">3</a></sup>


However, even without using that function Rust will no longer stop us from shooting ourselves in the foot. Can you spot the memory unsafety bug in the following function?


```rust
/// Remove a node from its parent.
/// This method will mutate the parent if it exists.
pub fn remove_from_parent(&mut self) -> Option<Node> {
    let mut maybe_node: Option<Node> = None;
    if let Some(mut parent) = self.get_parent() {
        if let Some(index) = parent.children.iter().position(|c| c == self) {
            maybe_node = Some(parent.children.remove(index));
        }
    }
    self.parent = ptr::null_mut();
    maybe_node
}
```

The bug is on the line where we set `maybe_node` to `Some(parent.children.remove(index))`. The `Node` we are moving into `maybe_node` is the same `Node` pointed to by `self` so after the removal `self` will be a dangling pointer. In practice, this means that when we set the parent to be `ptr::null_mut()` after the removal we are actually setting the sibling of the original `Node`'s parent since when we remove the `Node` the vector moves all of the elements over to the left.<sup><a href="#vector-unsafety-gist" name="vector-unsafety-gist-back">4</a></sup>


What about the next function? This one is more tricky to spot and is actually a bug with our `Vec`.

```rust
struct Node {
    parent: *mut Node,
    children: Vec<Node>
}

impl Node {
    fn add_child(&mut self, mut child: Node) {
        child.parent = self;
        self.children.push(child);
    }
}
```

When a `Vec`'s capacity is reached it is automatically reallocated to have double the capacity.<sup><a name="vec-src-back" href="#vec-src">5</a></sup>. When that happens, any references/pointers to the data in the original vector become dangling since every <code>Node</code> in the vector was moved to the new one. To fix this, you'd either have to change your data structure to something that doesn't reallocate when the capacity is reached (such as a linked list) or you have to constantly re-validate the sibling's parent pointers whenever you add a child to a `Node`.<sup><a href="#vector-resize-gist" name="vector-resize-gist-back">6</a></sup>

## Petgraph to the rescue!
Right now, there is no crate that provides a tree with all the features that we need for Way Cooler. Hopefully there will be one in the future, maybe one provided by us. Currently, Way Cooler uses [petgraph](https://crates.io/crates/petgraph) for a tree. We have an adapter module that maps graph operations to tree operations. In a way, it's just like using an adjacency list, and what a "proper" tree would probably look like in Rust. `Node` removals are still something that needs to be carefully managed, but there is no longer the possibility of a segfault in the tiling code for Way Cooler.

Though Rust is still a very new systems language, its benefits are already showing. Despite the immaturity of Rust's ecosystem, the existing tools are very versatile and well documented enough to work as good as, and perhaps even better than, an equivalent library in C. The library ecosystem is still very young and it's a shame that there is no reliable standard tree data structure on crates.io yet, but that should hopefully change in the coming years as Rust gains in popularity.

---
<p><sup><a href="#weak-parent-back">1</a></sup><a name="weak-parent"></a> Note that we can not have both the parents and the children be reference counted, otherwise that would create a cycle and consequentially a memory leak.</p>
<p><sup><a href="#smart-pointer-tree-code-back">2</a></sup><a name="smart-pointer-tree-code"></a> Here's a link to the <a href="https://github.com/Immington-Industries/way-cooler/blob/7e22199bd57e9286a74c98e2b829a0ace72ecadd/src/layout/containers.rs">full code</a>, if you'd like to see just how nasty it can get.</p>
<p><sup><a href="#as_mut_gist_back" name="as_mut_gist">3</a></sup>Play around with the <code>as_mut</code> bug <a href="https://is.gd/06U1FT">here</a></p>
<p><sup><a href="#vector-unsafety-gist-back">4</a></sup><a name="vector-unsafety-gist"></a> Play around with the inconsistent <code>self</code> bug <a href="https://is.gd/RKdp5B">here</a></p>
<p><sup><a href="#vec-src-back">5</a></sup><a name="vec-src"></a><a href="https://doc.rust-lang.org/src/alloc/up/src/liballoc/raw_vec.rs.html#212"> Rust source for Vec</a></p>
<p><sup><a href="#vector-resize-gist-back">6</a></sup><a name="vector-resize-gist"></a> Play around with the vector invalidation bug <a href="https://is.gd/OlbcFu">here</a></p>
