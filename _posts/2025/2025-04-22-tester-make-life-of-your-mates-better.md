---
title: Tester, make the lives of those around you better
layout: post
date: '2025-04-22'
last_modified_at: '2025-04-22'
categories:
  - development
tags:
  - testing
---

Let’s dig into the development process a bit. How does it usually look? First of all, the system analyst creates the documentation. The developer implements it, and then the tester validates the result. Looks pretty familiar, doesn’t it?
The trouble is that problems surface only at the end, when they are most expensive to fix.

<img height="500" title="TMNT in Java" alt="TMNT in Java" src="/assets/images/testatest.png">

## Don’t wait—act!
Don’t be lazy. If you see your developer taking on a new task, open it, find the analyst’s documentation, and start writing test cases. Imagine a robot reading your test cases and doing exactly what they specify.
The precision a robot demands is exactly what test frameworks require—so the habit transfers directly.
I believe this approach will help you transition to automated testing.
Tester judgment—spotting edge cases, usability gaps, and context-dependent behavior—still matters; the robot metaphor covers only the deterministic, specifiable part.



## Make calls your enemy
Whenever you’re about to call someone to clear up an ambiguity about a requirement or a bug, pause and ask yourself:

1. **What specific detail is missing** that would allow you to avoid the meeting?  
2. **Write that missing detail or question in text**, then add it as a comment to the task it belongs to:
   - If it’s a question about the documentation, find the task where the analyst wrote it and leave your comment there.  
   - If you’ve discovered a bug in the developer’s implementation, find the corresponding task and add a new bug report as a comment.  

> **Important!** Use markup and formatting (headings, lists, code snippets) to make your comments easy to read.



## How to write a bug report so that the developer will want to fix the bug
When a developer writes a test, they usually look at three things:

1. **What input data was provided?** – Include the data you used to test the implementation.  
2. **What data are expected?** – Share the expected data, response, or result.  
3. **What was the actual result?** – Include the actual data, response, or result to compare with the expected outcome.  

If you provide these details to the developer, the bug-fixing process will be significantly faster!



## What not to do
Avoid these things:

- **Asking the developer how things should work:** It’s your responsibility to know how a feature should work; if you don’t, how can you test it? If the documentation is missing or wrong, report that gap in writing on the relevant task before testing.  
- **Reporting bugs outside of the ticket:** Report bugs in the ticket, not in chat or verbally—informal reports vanish and cannot be tracked.  
- **Using vague phrases like “This does not work properly”:** ‘Properly’ is unclear—specify exactly what’s wrong.

Every one of these habits saves your teammates time they would otherwise spend in meetings, chasing vague reports, or re-explaining context—that is what makes their lives better.  
