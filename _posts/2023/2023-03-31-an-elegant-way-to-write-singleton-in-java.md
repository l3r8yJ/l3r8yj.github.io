---
title: An elegant way to write Singleton in Java
layout: post
date: '2023-03-31'
last_modified_at: '2023-03-31'
categories:
  - java
  - development
  - singleton
  - patterns
tags:
  - blog-post
  - development
  - singleton
  - java 
  - patterns
---

<img height="500" title="A hero of our time" alt="A hero of our time" src="/assets/images/hero-of-our-time.jpeg">

I just typed "The book about loneliness" into google, you should read it. I hope so. But if you're looking for a book to *become lonely*, you should rather look in the section of "software".

This blog post is dedicated to senior software developers with 5+ years of experience.

According to Wikipedia the Singletone is

> *"In software engineering, the singleton pattern is a software design pattern that restricts the instantiation of a class to a singular instance."*

We will not discuss whether Singleton is good or bad, for some reason we have to implement it in our code. This raises the question, how should we do it **right**?

First of all, let's look at the typical implementation seen in some training platforms, books, etc.

```asm
public final class Dummy {

  /**
   * Let's imagine that we have something heavy here.
   */
  private final String value = "I'm too heavy!";

  /**
   * Singleton instance.
   */
  private static final Dummy INSTANCE = new Dummy();

  /**
   * Private ctor.
   */
  private Dummy() {}

  public String value() {
    return this.value;
  }

  public static Dummy getInstance() {
    return Dummy.INSTANCE;
  }
}
```
<br/>

## Perfomance

Now it looks like we got what we wanted, 
but let's pretend that we only need this instance once in a while. We don't want to initialize it all the time to use the machine's resources. The obvious solution we have to make,
is to make our singleton [lazy](https://en.wikipedia.org/wiki/Lazy_initialization).

It should look like

```asm
public final class Dummy {

  /**
   * Now we are not instantiated before it is necessary.
   */
  private static Dummy INSTANCE = null;

  public static Dummy getInstance() {

  /**
   * Asking our Singleton – Hey, buddy, do you exist?
   */
  if(null == Dummy.INSTANCE) {
      
    /**
     * If it doesn't, let's create a new one.
     */
    Dummy.INSTANCE = new Dummy();
  }
    /**
     * If so, we'll return it.
     */
    return Dummy.INSTANCE;
  }
}
```
If we weren't in the real world, we could say that our code has no issues now. It's quite performant thanks to lazy initialization and we get a single object instance. But there's a big problem, our implementation **isn't thread-safe.**

<br/>

## Thread Safety

What does **not thread-safe** mean in our context? Just one thing, our Singleton is not exactly a singleton. *Because of the thread-safety problem, we can find cases where our object can be instantiated more than once*.

Let's take the case when two threads access our instance for the first time. 
Well, now **`#getInstance()`** is called, we are going to compare **`Dummy.INSTANCE`** with **`null`**. Both threads see that **`Dummy.INSTANCE`** is **`null`** and call a *new* instance.

At best we can expect one instance to overwrite the other, at worst we get two references to different objects. *Which will actually make our pattern not work!*

If you are slightly familiar with Java concurrency, you can suggest a solution that looks like this

```asm
public synchronized static Dummy getInstance() {
  if (null == Dummy.INSTANCE) {
      Dummy.INSTANCE = new Dummy();
  }
  return Dummy.INSTANCE;
}
```
And indeed, such a solution has the right to life, but **we have problems with performance again**...

If we take a deeper look at this case, we see one thing. We don't want to synchronize every time we call **`#getInstance()`**. All we have to do is to check if the object exists, if we synchronize the **whole** method then we get essentially a queue of threads, which is no longer a multithreaded execution. Here you can see where the performance problem comes from.

Now you can provide me solution like that

```asm
public static Dummy getInstance() {
  /**
   * Checking when we aren't synchronized.
   */
  if (null == Dummy.INSTANCE) {
      synchronized (Dummy.INSTANCE) {
          /**
           * Another synchronized check in case another thread 
           * initialized the instance during our earlier check.
           */
          if (null == Dummy.INSTANCE) {
              Dummy.INSTANCE = new Dummy();
          }
      }
  }
  return Dummy.INSTANCE;
}
```

It's really very close to what we need, but we haven't noticed a single thing. It's the "[happens-before](https://stackoverflow.com/a/11970581/11529150)" relation. In fact, it's not the easiest topic, so I'm just providing a link to it. 

To satisfy the visibility rule for this relation, we should use the keyword **`volatile`**.


```asm
public static Dummy getInstance() {

  private static volatile Dummy INSTANCE = null;

  /**
   * Checking when we aren't synchronized.
   */
  if (null == Dummy.INSTANCE) {
      synchronized (Dummy.INSTANCE) {
          /**
           * Another synchronized check in case another thread 
           * initialized the instance during our earlier check.
           */
          if (null == Dummy.INSTANCE) {
              Dummy.INSTANCE = new Dummy();
          }
      }
  }
  return Dummy.INSTANCE;
}
```

In the end, we can say that we now have a thread-safe implementation of the Singleton pattern.

But I want to ask you one question. 

> "*Do you find this solution elegant?*"

Honestly, I don't.

<br/>

## Elegant way is

In my opinion the best way to implement Singleton is

```asm
public class Dummy {

  private final String value = "I'm too heavy!";

  public String value() {
    return this.value;
  }

  static Dummy getInstance() {
    return Dummy.Holder.INSTANCE;
  }

  /**
   * Lazy.
   */
  private static class Holder {
    static final Dummy INSTANCE = new Dummy();
  }
}
```

This is the complete solution, it is the Singleton pattern specification.
We're still lazy, which means we're still **performing**, and we're **thread-safe** because we're **immutable**!

This approach is called *initialization on demand holder idiom.*

<br/>

Thanks!

<br/>
