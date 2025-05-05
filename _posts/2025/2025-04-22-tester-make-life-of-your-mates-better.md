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

<img height="500" title="TMNT in Java" alt="TMNT in Java" src="/assets/images/testatest.png">

## Don’t wait—act!
Don’t be lazy. If you see that your developer is taking on a new task, open it, find the analyst’s documentation, and start writing test cases. Imagine a robot reading your test cases and doing exactly what they specify. I believe this approach will help you transition to automated testing.



## Make calls your enemy
Whenever you’re about to call someone on the team to clear up an ambiguity, pause and ask yourself:

1. **What specific detail is missing** that would allow you to avoid the meeting?  
2. **Write that missing detail or question in text**, then add it as a comment to the task it belongs to:
   - If it’s a question about the documentation, find the task where the analyst wrote it and leave your comment there.  
   - If you’ve discovered a bug in the developer’s implementation, find the corresponding task and add a new bug report as a comment.  

> **Important!** Use markup and formatting (headings, lists, code snippets, etc.) to make your comments easy to read.



## How to write a bug report so that the developer will want to fix the bug
When a developer writes a test, they usually look at three things:

1. **What input data was provided?** – Include the data you used to test the implementation.  
2. **What data are expected?** – Share the expected data, response, or result.  
3. **What was the actual result?** – Include the actual data, response, or result to compare with the expected outcome.  

If you provide these details to the developer, the bug-fixing process will be significantly faster!



## What not to do
Avoid these things:

- **Asking the developer how things should work:** It’s your responsibility to know how a feature should work; if you don’t, how can you test it?  
- **Reporting bugs outside of the ticket:** Avoid informal bug discussions, as they waste time for you and your team.  
- **Using vague phrases like “This does not work properly”:** ‘Properly’ is unclear—specify exactly what’s wrong.  
