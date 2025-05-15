---
title: IU-chain testing
layout: post
date: '2025-05-15'
last_modified_at: '2025-05-15'
categories:
  - development
tags:
  - testing
---

> Every developer has sat through a 20 minute build, wondering which of a hundred tests just broke—only to discover it was ‘that one obscure component’

Not that long ago, I was taking a bath and, as often happens, an idea came to me.
I don’t know what to call it, so let’s call it ***“Integration‑Unit Chain Testing”***.
I just want to share this idea, which I believe might be useful.

## Problem of build time, simplicity of localization, and reliability

The main goal of this approach is to reduce TTM (time to merge) and speed up bug localization.
Let’s imagine we have a few integration tests (interpret “integration test” however you like—I’m just sharing the idea):

```asm
@Test
@Chain("LeaderIntegrationTest")
fun `checks something important`() {
    val request = ...
    val response = requestExecutor.send(request)
        .response()
    assertThat(response).isOkay()
}
```

As far as I know, integration tests are designed to validate a large component as a “black box” (if you’re not using mocks—and I think you shouldn’t!):

```asm
 ______________________________________
|---Big unit for integration testing---|
|--------------------------------------|
|componentA -> componentB -> componentC| 
|--------------------------------------|
```

When this big integration test fails, it’s often hard to determine which component is broken.
What should we do *as professional programmers*? Write a *unit* tests untill we will reduce the scope and find broken component!
Since we’re writing these tests, the total number of tests keeps increasing.

## More tests equal to longer build time

Whenever we run the full build, we have to wait for every test to complete.
At that point, I only want to run the integration tests first—and if they pass, skip the rest, since everything “just works.”
But what happens when one of these integration tests fails? As I said before, we need to determine which component let us down. With a large suite of unit tests, it becomes much easier: we can identify the failing component by seeing which unit test fails.
With this approach we retain both the *reliability* of integration tests and the *localization* of unit tests, but at the cost of *build time*.

Here’s how those trade‑offs typically look:

```asm
+------------+-------------+---------------+
| build time | reliability | localization  |
|     1      |      1      |       0       |
|     1      |      0      |       1       |
|     0      |      1      |       1       |
+------------+-------------+---------------+
```

It seems difficult to optimize all three at once.

## Check slow, localize fast!

To have the best of all worlds, I suggest *chaining* integration and unit tests together.
The main idea is to run the “leader” integration test first and trigger its linked unit tests only when it fails:

```asm
// "main" integration test
@Test
@Chain("LeaderIntegrationTest")
fun `checks something important`() {
    val request = ...
    val response = requestExecutor.send(request)
        .response()
    assertThat(response).isOkay()
}

// chain of unit tests behind it
@Test
@ChainLink("LeaderIntegrationTest")
fun `unit test for componentA`() {
    // some test
}

@Test
@ChainLink("LeaderIntegrationTest")
fun `unit test for componentB`() {
    // some test
}

@Test
@ChainLink("LeaderIntegrationTest")
fun `unit test for componentC`() {
    // some test
}
```

> \[!IMPORTANT]
> This is a concept, not a rule. You can enable it only in CI, while on your machine you can still run all the tests.

In the end, we can envision something like this:

```asm
|integration test A:
|-----------------+
|                 |unit test 1
|                 |unit test 2
|                 |unit test 3

|integration test B:
|-----------------+
|                 |unit test 1
|                 |unit test 2
|                 |unit test 3

|integration test C:
|-----------------+
|                 |unit test 1
|                 |unit test 2
|                 |unit test 3
```

Here, our integration tests act as gates and the unit tests as guards. Gates block the bugs, and guards help us locate where they’re hiding.
