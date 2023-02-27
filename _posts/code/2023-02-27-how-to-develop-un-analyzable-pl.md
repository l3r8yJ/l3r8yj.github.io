---
title: How to develop un-analyzable PL 
layout: post
date: '2023-02-27'
last_modified_at: '2022-02-27'
categories:
- static analysis
tags: blog-post
---
### Introduction

So, today I want to tell you how you should develop a programming language that will never surrender to a static analyzer. After all, we all know that static analysis is bad, quality control is for those who cannot write clean code, and all unit tests should be written by junior programmers! If you agree with at least one point, write me an e-mail, or better in comments.  I must convince myself that such people exist.


A small disclaimer for those who are against static analysis and code quality control tools. 
  > It is [proved mathematically](https://www.cs.virginia.edu/~robins/Turing_Paper_1936.pdf) that static analysis can never be 100% true. *This is impossible to write a program that understands how another program will execute without first executing it.*

However, this **does not** mean that static analysis is useless or that it cannot be effective in finding bugs and vulnerabilities in software. 

<br/>

On a slightly more serious note, what makes a programming language *difficult to analyze*?

<br/>

### Syntax sugar

Let's take a look at syntax sugar, how it's usually looks?

Here a Java example:

```java
...
var post = someVariableDeclaredBefore;
...
```

Things like `var` on the one hand help the programmer write code shorter than it can be, and then in Runtime a wise compiler will put in the missing pieces of code, and oh miracle, everything is fine.


How does a static analyzer resist this? Here we have one option - to analyze compiled code. In java, for example, these are `.class` files.


This approach has many disadvantages. For example, the first one is that the analysis won't be *smooth* as if we were doing it with the source code. The second one is that it's very difficult or impossible to write plugins for the IDE. You also are unable to analyze comments.

But you're not much bound to the version of the language you're analyzing, as if you were analyzing the source code. And you don't have to parse source code files as you often do with *third-party* tools.

<br/>

So, do you remember this? Now add as much syntax sugar to your language as you can!

<br/>

## Massive AST

This point is very easy to understand, because the bigger your [AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree) is, the harder it is to parse, lexify, etc. Imagine all these giant constructions embedded in each other, sometimes it's just hard to understand what's written here. But try to analyze it...

<br/>

So you should increase your AST to [Statue of Unity](https://en.wikipedia.org/wiki/Statue_of_Unity) size.

<br/>


## Mutability

Well, when I talk about mutability in this context, I mean reassigning the same variable multiple times with parallel reading from it. The difficulty of analysis here lies not only in the equal sign.

```java
File f = new File("foo.txt");
f.doSmth();
f.setPath("bar.txt");
f.writeSomethig(content);
```
Changing statements multiple times makes analysis difficult, because keeping track of execution flows becomes a complex task. We must teach our tool to remember when and how each statement changes.

If we can't create setters, the code might look like this:
```java
final File f1 = new File("foo.txt");
final File f2 = new File("bar.txt");
f1.doSmth();
f2.writeSomethig(content);
```
Now, for each file we have only one way of sequence
This piece of code is much easier to analyze, isn't it?

<br/>

Now you know that you need to ban immutability in your artwork from the world of programming languages.

<br/>

## Multiple conditional flow

How often you see something like that:
```java
String someCoolMethod(String unsafe) {
  // code above
  if (unsafe.isEmpty()) {
    return safeVariable;
  } else {
    return variableThatDependsOnUnsafeString;
  }
}
```
Here we see that we can rely on this method only for cases from the first branch. The math here is simple:
`safe behavior + unsafe behavior = unsafe behavior` *(1)* – it's called *merging*. Why the code I cited above is problematic. Let me explain. Imagine we don't use *merging* and the method is called multiple times in a row.

Let it be called 5 times. How many conditional flows will be produced?
If you're an optimist, you said `10` perhaps. If you know math a little better, you probably said something like: *the number of possible branches is the base of the degree, and the number of challenges is the exponent of the degree*. And you were right.

The answer is `2^5 = 32` conditional flow you have to analyze. But, what if the method you are analyzing has about `10` conditions with only `2` branches? Yes, that's **1024 condition flows**. How does a static analysis tool learn to do this?
It's easier to say we have a vulnerability. Than to keep all possible variations in mind.

If we merge each branch into one using the formula *(1)*. With the previous condition, when we have `5` times the call, we get `5` conditional flows.

I think you can still familiarize yourself with the concept of [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity).

<br/>

So do not forget this when you create your language. Make it impossible for the compiler to compile unless a method contains at least three, and preferably four, `if` branches within it.

<br/>


## Polymorphism

What you can say about this qoute?
  > "*The goal of polymorphism is essentially the opposite of static code analysis.*"
  >
  > &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; – Arno Haase

Before I can explain it to you, let's look at this piece of code.
```java
interface Request {
  Response act();
}

class RqSafe implements Request { 
  @Override
  Response act() {
    // safe implementation
  }
}

class RqUnsafe implements Request { 
  @Override
  Response act() {
    // unsafe implementation
  }
}

public List<Response> someProcessingMethod(final Collection<Request> reqs) {
  return reqs.stream()
    .map(r -> r.act()) // boom!
    .collect(Collectors.toList());
}
```

What should we do here? How can we easily check which implementation of the `act()` method is safe and which is unsafe? What to do if we can only see contracts from interfaces. 

Imagine if the library you're using only contains an open interface and you don't know what specific implementation will be used in the code you write? How easy is it to analyze?

Polymorphism is about making it hard to figure out what is going on inside. When static analysis is about finding out and knowing exactly what it does when we call a method. This is a big problem for static analysis tools to figure out.

*Please don't misunderstand me. I'm **not** saying that polymorphism is a bad thing. I'm saying that it makes the code **more difficult** to analyze.*

How to resist polymorphism? We can use [Call Graph](https://en.wikipedia.org/wiki/Call_graph). This is just a list of possible targets for each function call statement. We need to look at the `act()` invoke and figure out which implementations it can go to. We do not know what was placed inside the `reqs` parameter. Therefore, in our case it will be both implementations, `RqSafe` and `RqUnsafe`.

<br/>

Going back to your brand new programming language. Given the above, you need to make it impossible to write implementations within a single module.

<br/>


## Conclusion

Here I've listed a few things that make static analysis tricky, I'm sure it's not the whole list, but I found these things to be quite interesting. As I said before, static analysis **can't be right**. So when we find a vulnerability, it's hard to present the correct results. Here is the dilemma between the two states:
 
 1) Report anything we can find.

 2) I'm not sure I need to report it.

It's all about trade-off. 


<br/>

Thank you, I hope this post was interesting for you, also you can correct me in the comments if I made mistakes, etc.


