---
title: Don't let warnings to poison you
layout: post
date: '2024-06-26'
last_modified_at: '2024-06-26'
categories:
  - development
tags:
  - building
  - CI/CD
  - linter
  - project-management
---

A few days ago I ran into a situation where we decided to start rejecting some code using linter,
the decision has been made, pull request is open, and all that's left is to merge these changes.
But the rule that was added is marked as a _warning_. This fact confused me a little bit, that's
why...

So, there is an original
link – [github.com/fakehub/pull/89](https://github.com/h1alexbel/fakehub/pull/89#pullrequestreview-2190111637)

<img width="500" title="Definitely" alt="Definitely" src="https://imgs.xkcd.com/comics/definitely.png">

## Purpose of linter

In my opinion, linter, which is built into CI/CD pipeline, has several goals. _One of them – be a
teacher for programmer_.
In moment when you're setting up the pipeline, you're adding specific rules that come from your
experience.
You're describing how code should be written in this project
and thereby you're _sharing_ your expertise, knowledge,
best practices, etc. through CI/CD.
Coming back to warnings, it means that
_warnings create uncertainty by pointing out an error and allowing the build to succeed_.
I mean, it's like we're saying this is bad, but we're going to let you merge pull request, which is
pretty controversial, isn't it?
And the programmer in your project continues to write the code that generates these warnings,
which means that he will just get used to writing code that smells in your project.