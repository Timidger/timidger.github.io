---
layout: post
title: "RiiR considered harmful"
date: 2019-04-10
categories: blog
visible: 1
---

This paper was originally written for a class while I was attending University.

It was written about the Rust programming language and talked about an issue
that I have not seen gain traction anywhere else. It posits that rewriting
libraries that are originally written in C to Rust (RiiR) is a bad engineering
decision and that instead programmers should strive to write bindings to the C
library.

I recognize that this is a difficult undertaking due to how Rust is designed to
be memory safe. I give examples of well done C binding libraries, such as rlua,
and talk about my experiences with wlroots-rs, which are bindings to the wlroots
library that I wrote my self.

Since I wrote this piece I have had a slight change of heart. C bindings are the
right thing to do, but they are so horribly boring and difficult they are not
worth doing. If you are writing a program and you think Rust is a good fit, it's
only a good fit if the libraries you want to use are wrapped or implemented in
Rust. If bindings or a RiiR library does not exist, then you are better off just
using C. Maintaining bindings for any non-trivial C library is a full time job
and usually tangential to the goal you actually want to achieve.

Personally speaking, I don't think that something is objectively better just
because it's written in Rust. That's not a feature to me, just an implementation
detail, and undue effort should not be done just so that Rust can be used if it
is not going to make the program better.

I will be making a bigger blog post about this later as this will affect my
design of Way Cooler and have ramifications for wlroots-rs.

[RiiR Considered Harmful: Just Bind to C
Libraries](../riir-considered-harmful.pdf)
