---
layout: post
title: The Cost of Moving Fast
date: 2018-05-11
category: blog
---

Rust [recently released version 1.26](https://blog.rust-lang.org/2018/05/10/Rust-1.26.html) and with it a big change that has been in the pipeline quite a while: [impl Trait syntax](https://github.com/rust-lang/rust/issues/34511).

I won't go into the change itself (as it's a little involved, especially if you don't know Rust) and I won't give my opinion of the change because I haven't actually tried it yet so I don't know if it's a net win for the language yet.

What I'm going to talk today is the programmer overhead resulting from changes like this.

## The (changing) Language Landscape
Most languages have and will continue to change.<sup><a href="#most-languages" name="most-languages-back">1</a></sup> You have different amounts of change at different rates of change. 

If you plotted them, you'd find that C is on one far end of the spectrum (They come out with a standard basically once a decade, and the changes are pretty small for the most part) and Rust being way on the other (with a new minor release every 6 weeks).

When you write some code in C, assuming the business needs don't need to change, you can assume it will still be just as idiomatic ten years after you write it. That's not to say there's one right way™, of course, but compared to other languages the quality of C code isn't traced with what language features the programmer used but rather by the standards of engineering that were in place at the time (and place) the code was written.

In other languages however, you will find that there's "old code" that, while not incorrect architecturally or logically, is using old "bad" features. Not using context managers in C# or Python is an example of "old code". The version that has these shiny new features is still backwards compatible with that old "bad" code. This is often touted as a feature.

Often time that backwards compatibility is broken, often to the detriment of companies still using that old "bad" code. The Python 2/3 split is an obvious example of this. A less obvious example is C and C++.<sup><a href="#c-is-not-c++" name="c-is-not-c++-back">2</a></sup> These big changes are where the most API churn happens and thus are thankfully several years apart to allow programmers to catch up.

## Rust and API Stability
The Rust team has done a tremendous job in sticking to their backwards compatibility policy. I personally have never had a build break because rustc or cargo updated because any backwards compatible changes are communicated very loudly via warnings and are few and far between.

What _has_ happened though is that "idiomatic" Rust has been a constantly moving target. Which is very frustrating as a programmer with not a lot of time after work.

Part of this is due to the language still being new and thus there needs to be time until patterns emerge and spread across the community to form an "idiomatic" style. Already there's very good practices that are mostly adhered to, the biggest one being only taking ownership when it's necessary and using `&[]` instead of `&Vec` and `&str` instead of `&String`. There's still issues with unsafe crates and their guarantees, but that's a hard problem and not really affected by what I want to talk about today.

With the latest change however lots of libraries have now become "old" code because they don't use the `impl Trait` syntax. They may still compile, but no one would really write it with the old style because it's just gross and the new style makes a much nicer API<sup><a href="#impl-trait-is-bad" name="impl-trait-is-bad-back">3</a></sup>.

This may not seem like a huge problem when viewed in the micro, but it becomes a huge problem when viewed in the macro.

### A fledgling ecosystem constantly kneecapped
Quite a few Rust crates are not 1.0 stable. This is a problem that has been addressed before, but it's made much worse by the constant API churn. The Rust developers are good at sticking to semvar, but crate maintainers are not. Even when they are, if you are below 1.0 you can make any breaking change you want in a minor bump.

When the `?` syntax stabilized most crates flocked to it, leaving the ones that didn't forced to update their compilers and stick with their "old" code. And this is a simple, non API breaking change. On the other side you have things like lack of NLL, which requires restructuring your code just to get it to compile and is sorely needed, and `impl Trait` syntax, both of which causes huge changes how Rust code is written and, once released, will need the old crates to either update their minimal compiler versions when their dependencies leave them behind and (eventually) rewrite their "old" code to fit this new standard.



---
<sup><a href="#most-languages-back" name="most-languages">1</a></sup> Unless you're talking about COBOL of course.


<sup><a href="#c-is-not-c++-back" name="c-is-not-c++">2</a></sup> C is **not** a subset of C++. They are entirely different languages. As well, "idiomatic" C++ changes depending on what version you are running (pre C++11 vs C++11 vs C++17).


<sup><a href="#impl-trait-is-bad-back" name="impl-trait-is-bad">3</a></sup> I actually have issues with `impl Trait`. As a return value it makes it hard to store in a struct without using polluting generics (or Boxing, which is what we are trying to avoid). Furthermore, using it in parameter variables is objectively worse because you lose the turbofish syntax.
