---
layout: post
title: Big Programming
categories: blog
visible: 0
---

Recently I was talking with a student I was TAing and he asked me some questions that, while not about the PL theory course he was taking, were questions that I had heard before. They were questions I had asked myself, and I'm sure many others had as well.

These questions were varied and targeted many things, but they can be boiled down into one question: "How do I make a big program?".

"Big" here is a little ambiguous. Big as in literally big? Big as in popular? Big as in complex? Sure maybe, but none of that is necessarily the _goal_ here. <sup><a href="#not-the-goal" name="not-the-goal-back">1</a></sup> "Big" here is a bad word, because the word he actually meant was "useful". He wants to make some useful software and it just so happens that all of the useful software out there is big.<sup><a href="#clarify-useful-small" name="clarify-useful-small-back">2</a></sup>


So how do you make ~~"big"~~ useful software?

# Making ~~"Big"~~ Useful Software
You make useful software by trying (and failing) to make big software. Repeatedly. Along the way you fail less and less and your big software eventually becomes useful software. You'll always fail eventually, bitrot affects all but the simplest of programs. But it will get easier.

For your first useful software you should take things slow though. Chances are, the thing your are making will not become the next Facebook or Linux. Instead, you should try to focus on what engineering skills you want to learn from this endeavor, rather than the software itself. You need to set goals, things you want to learn.

## Mao (or, How a Chinese Dictator Helped Me Learn OOP)
As an example, when I was started out the first difficult thing I had to learn that a textbook couldn't properly teach me was [OOP](https://en.wikipedia.org/wiki/Object-oriented_programming). To accomplish this goal, I decided to try to implement a little game called [Mao](https://en.wikipedia.org/wiki/Mao_(card_game)).

**Deciding to implement Mao was profoundly important**. Not because it went anywhere. You can see the fruit of my labors [here](https://github.com/Timidger/Mao), to my great shame. I probably re-wrote it 4 or 5 times and it still has a lot of bugs in it. It's not really usable, but that's not really the point. I had a lot of fun working on it because I really liked the problem. I really wanted Mao to be computerized because that seemed cool to me. It's very important that you choose something _fun_.<sup><a href="#not-too-fun" name="not-too-fun-back">3</a></sup>

I reached my goal because it was fun. In fact I exceeded my goal, probably because I was having so much fun. Not only did I learn about OOP, I learned about [basic networking](https://docs.python.org/3/library/socket.html), [multi-threading](https://docs.python.org/3/library/threading.html), and [GUI programming](https://docs.python.org/3/library/tk.html).

So pick something like that, some part of programming you're not very confident about or haven't ever touched. Then, find something fun you want to implement.


## Examples of Big Software
Oddly enough, this seems to be a big sticking point with a lot of programmers. If you can't think of anything fun, [here](/assets/challenge-list-1.jpg) are [some](/assets/challenge-list-2.png) lists [courtesy](/assets/challenge-list-3.png) of [/g/](https://4chan.org/g). Not all of those are great, but they should get you thinking.

My recommendation is to make a file manager. Though that might just be because all the ones I've been using are kind of crap, it's actually got quite a bit going on:
* Graphical Programming
* Lots of optional features you can tack on
* Useful software you use everyday


# How to Start
Okay, assuming you chose a nice project from the previous section, how do you start? Well first you have to choose your tools.

## Language
Choose the one you know best. It doesn't matter if it's dynamically/statically weakly/strongly typed. Any language will do, even ones designed in a week by a homophobe.

## Editor
Choose one you know best. If you don't know one very well, I suggest something easy like Atom, Sublime, or Visual Studio Code.

Eventually you should look into using Vim or Emacs. If you are up for a challenge you can look into those now but don't continue using them if it interferes with the project.

## Version Control
If you have never used distributed version control before (e.g Git or Mercurial) then I highly suggest you try out using git. Not only will it help you track your progress, you'll learn about a great tool you can use later.

If it seems like too much and it's going to put off you starting your project put it off until later. But definitely come back to this because this is a very important tool to learn.

## Libraries / Frameworks
Though this is somewhat project dependent (and my personal opinion), I highly suggest you don't use a large, complicated library/framework unless it's pivotal to what you need to do (and even then, consider something small).<sup><a href="#frameworks" name="frameworks-back">4</a></sup>

As an example, in Mao I didn't use anything that wasn't in the standard library of Python. Now it has a large standard library, but easily the most complicated thing I used was the GUI toolkit. Everything else was just a wrapper around some system call.

If you are making a GUI, by all means use GTK or QT or whatever. If you are making a website, consider using a small framework so you are actually learning the process of making that type of application rather than a framework (e.g Use flask, don't use django. Use vanilla JS, don't use React or Angular, etc.)

You can learn a lot by re-inventing the wheel. Now is the time to do it.

## Operating System
Whatever you're most comfortable with. Windows, OSX, Linux, whatever.

Seriously, I love Linux, but everyone loves dumping that on new programmers and it's really annoying. 

**You should absolutely learn how to use Linux**. 

_Just not right now_. 

Right now you're learning to make Big Software, and that doesn't necessitate learning Linux.


# Writing the Code
I assume you know how to write code, so I won't bother linking tutorials for that.

## Organizing Your Code

But you may not know how to organize it. That's ok, but something you'll need to solve later. I suggest reading literature on this. Some good books for this are [Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882), and my personal favourite [A Pragmatic Programmer](https://www.amazon.com/Pragmatic-Programmer-Journeyman-Master/dp/020161622X).

This is also going to be dependent on your language. A Java project is going to be structured very differently compared to a C project.

In the end though, don't worry about this at the start of the project. Start throwing things at the canvas, since you can always move things around later.


## Refactoring
For a big part of a big project is refactoring. If you write some code and then realize later that it's not designed well you should change it. This isn't always the case in _very_ big projects, but for these pedagogical exercises it's the best time to do heavy refactoring. 

I rewrote Mao _4 times_. Each time I learned a lot more about how to structure my code. Don't be afraid to do a full rewrite.

## Make Things Configurable
This depends on the application you're building, but making things configurable exposes you to a lot of new things. It means you have to choose a configuration file format, find a way to parse it, and generally make your code more flexible.

For some basic formats, I suggest using INI or, if that is too simple, YAML.

## Documentation
Writing documentation is a skill that is hard to learn. I still think it's something I'm bad at. The only way to get better at is practice.

Documentation doesn't just refer to doc comments. It also means writing READMEs, writing high level design documents, and writing git comments. For a good tutorial on how to read good git comments, see [this](https://chris.beams.io/posts/git-commit/).

## Make It Easy to Test
You should make it easy to test your code. I don't just mean easy to set up and run automated tests against it (though you should do that too).

 You should be able to, with a single command/click, run the program in order to test a new feature out. Anything more than that and it makes it harder to iterate on your programmer. This feedback loop is very important as it reduces friction and helps you concentrate on your work without breaking flow.

# Takeaways
## Setting Sub Goals
One of the biggest things you can do to improve your chances of success is setting sub goals.

If you are writing a file manager, a good first sub goal would be to list the files in a directory on the console. Then to be able to have a "current" directory that can interactively print out the current contents. Now you should be able to print out files and folders differently. Now you want to list information about the files and folders (what type are they? How big are they?). 

This console is starting to become limiting. Now maybe you'll want to look into a TUI or GUI library to display files and folders. Then you should add icons. Maybe a tree structure on the left would be helpful. A search bar would be pretty cool. Ooo, what if the user wants to set a theme? You gotta add that feature.

If you start with these small goals not only does it make it easier to stay motivated (because you get the satisfaction when you reach that goal) and set up an easy TODO list for you tackle, but it also encourages you to think about and get excited for the features you will add to your program.


## Show People
Show people your work. It can be your friends and family or it can be other programmers. By showing other people you can feel good about the thing you accomplished.

 Don't be afraid of criticism, as long as it's valid criticism. If it's objective and about the work, use it to improve it. If it's ad hominem and mean, ignore them.

## Knowing When to Stop
Eventually, a project will stop being useful for you to work on. Maybe it will just get boring, or maybe any more value you can extract from it isn't worth the effort. Don't be afraid to stop before it's "complete". Remember what the ultimate goal here is: learning. If you "fail" but learn something, you didn't actually fail.

## Further Reading
* [What every computer science major should know (Matt Might)](http://matt.might.net/articles/what-cs-majors-should-know/)
* [Programmer Competency Matrix](http://sijinjoseph.com/programmer-competency-matrix/)
* [Codeless Code](http://thecodelesscode.com/contents)
* [Worse is Better](https://www.dreamsongs.com/WorseIsBetter.html)
* [The Architecture of Open Source Applications](http://aosabook.org/en/index.html)
* [Ian's Shoelace Site](https://www.fieggen.com/shoelace/)


---
<sup><a href="#not-the-goal-back" name="not-the-goal">1</a></sup> What you actually want to make some _popular_ software? It's fun to have people use your stuff. But if you've never made big software before, lets hold off on that goal. Besides, perhaps that's not really a good goal to actually have when designing software.


<sup><a href="#clarify-useful-small-back" name="clarify-useful-small">2</a></sup> Yes, yes, yes. Small scripts can be useful. That's not what I'm talking about either. They are only useful within the context of "big" programs too. I have a small script I have that controls the brightness of my screen. But it's no more useful than simply typing `echo $(($(cat /sys/class/backlight/intel_backlight/brightness) + 10)) > /sys/class/backlight/intel_backlight/brightness` except for saving a few keystrokes. You know what _is_ useful? Hooking that up to activate on a keyboard press using the window manager I wrote.

<sup><a href="#not-too-fun-back" name="not-too-fun">3</a></sup> But don't choose something _too fun_. A video game is very fun. If you want a good one check out [this one](http://stabyourself.net/nottetris2/). But games and things like that are very challenging and easy to get burnt out on. It's better to aim too low when you're starting out rather than shoot too high and burn yourself out.

<sup><a href="#frameworks-back" name="frameworks">4</a></sup> Large libraries and frameworks are really helpful, especially when working on a large team. However, libraries come and go and it's much important to learn fundamentals that stick around first like POSIX, language specs, and web standards.
