---
title: Tester, make the lives of those around you better
layout: post
date: '2025-04-22'
last_modified_at: '2026-10-09'
categories:
  - testing
tags:
---

Let’s dig into the development process a bit. How does it usually look? First of all, the system analyst creates the documentation. The developer implements it, and then the tester validates the result. Looks pretty familiar, doesn’t it?
The trouble is that many problems surface only at the end, when fixing them means reopening finished work. And the way testers communicate decides how much of everyone else's time each of those problems eats.

<img height="500" title="TMNT in Java" alt="TMNT in Java" src="/assets/images/testatest.png">

## Don’t wait—act!
Waiting for the build costs the team time: every question you could have asked earlier now arrives after the code is written. If you see your developer taking on a new task, open it, find the analyst’s documentation, and start writing test cases. Imagine a robot reading your test cases and doing exactly what they specify.
Requirements may still change while the developer works, so keep these early cases at the level of the requirements—what goes in and what should come out—rather than clicks and screens; they are cheap to adjust, and every gap they expose is a question answered before the code is written.
The precision a robot demands is what test frameworks require too—so part of the habit transfers: breaking a check into exact, deterministic steps.
I believe this approach will help you transition to automated testing.
Tester judgment—spotting edge cases, usability gaps, and context-dependent behavior—still matters; the robot metaphor covers only the deterministic, specifiable part.



## Write first, call last
Whenever you’re about to call someone to clear up an ambiguity about a requirement or a bug, pause and ask yourself:

1. **What specific detail is missing** that would allow you to avoid the meeting?  
2. **Write that missing detail or question in text**, then add it as a comment to the task it belongs to. If the text doesn’t settle it, call—then record the outcome on the same task:
   - If it’s a question about the documentation, find the task where the analyst wrote it and leave your comment there.  
   - If you’ve discovered a bug in the developer’s implementation, find the corresponding task and add a new bug report as a comment.  

> **Important!** Use markup and formatting (headings, lists, code snippets) to make your comments easy to read.



## How to write a bug report so that the developer will want to fix the bug
When a developer writes a test, they usually look at three things:

1. **What input data was provided?** – Include the data you used to test the implementation.  
2. **What data are expected?** – Share the expected data, response, or result.  
3. **What was the actual result?** – Include the actual data, response, or result to compare with the expected outcome.  

For example, compare “Login does not work properly” with:

> **Input:** `POST /login` with `{"email": "user@example.com", "password": ""}`  
> **Expected:** `400` with message “Password is required”  
> **Actual:** `500 Internal Server Error`

The second report can be turned into a failing test right away; the first one starts a conversation. Give developers something they can reproduce in a minute, and fixing it becomes the easy part.



## What not to do
Avoid these things:

- **Asking the developer how things should work:** It’s your responsibility to know how a feature should work, and the documentation is where that knowledge should live. If the documentation is missing or wrong, report that gap in writing on the relevant task before testing.  
- **Reporting bugs outside of the ticket:** Report bugs in the ticket, not in chat or verbally—informal reports are hard to find later and aren’t linked to the work item.  
- **Using vague phrases like “This does not work properly”:** ‘Properly’ is unclear—specify exactly what’s wrong.

Every one of these habits saves your teammates time they would otherwise spend in meetings, chasing vague reports, or re-explaining context—that is what makes their lives better.  
