---
title: A proper understanding of encapsulation.
layout: post
date: '2023-02-20'
last_modified_at: '2023-02-20'
categories:
- oop
- encapsulation
tags: blog-post
---

![pic](https://media.licdn.com/dms/image/D4E12AQGvsgwIo0TIQQ/article-cover_image-shrink_423_752/0/1662032433004?e=1682553600&v=beta&t=3C_8V2XIMeBd6rNX4kHLhGr5cdnfZJLnVhgm9EEWVwk)

## Let's see...

I remember when I started studying OOP and came across the definition of encapsulation, it looked like this:


> *In OOP, encapsulation refers to the bundling of data with the methods that operate on that data, or the restricting of direct access to some of an object's components. Encapsulation is used to hide the values or state of a structured data object inside a class, preventing direct access to them by clients in a way that could expose hidden implementation details or violate state invariance maintained by the methods.*


I was confused by the definition...

And based on this definition, it became clear to me as a novice programmer that **encapsulation is equal to hiding object data**.

Doesn't such an interpretation of this definition seem wrong?


<br/>
## Let's look at an example of code in Java, since Java is one of the "object-oriented" languages.
<br/>
*Here we see the `Coefficient` class used to write the quadratic equation:*
<br/>
```java
public class Coefficient implements EquationElement {
  
  private final double value;

  public Coefficient(final Object value) {
    this.value = (Double) value;
  }

  @Override
  public double value() {
    return this.value;
  }

}
```

<br/>
*Here we see the class of the `Discriminante` used to solve the quadratic equation:*
<br/>
```
public class Discriminante implements EquationElement {
  
  private final Coefficient a, b, c;

  public Coefficient(
    final Coefficient a,
    final Coefficient b,
    final Coefficient c
  ) {
    this.a = a;
    this.b = b;
    this.c = c;
  }

  @Override
  public double value() {
    return Math.pow(b.value(), 2) - 4 * a.value() * c.value());
  }

}
```
<br/>
They both implement the `EquationElement` interface:
<br/>
```java
public Interface EquationElement {
        
  double value();

}
```

As we see, both classes implement the value() method of the implemented interface and encapsulate the object's state. However, the implementation within the Coefficient class provides direct access to the data, while the implementation of the Discriminate class does not give the data directly, but modifies it in a certain way.

<br/>

So we see that in one case there is no hiding of data, and in the other case it just changes. Here we can get a more correct definition of encapsulation:

> *Encapsulation is a principle which allows us to combine data and methods which work with that data into a single object and **to hide implementation details** from the user.*

## Summary
That is, the definition that says that encapsulation is concealment is wrong, but it is also wrong to say that there is no concealment. Concealment is a side effect of encapsulation. *(By the way, concealment is violated by setters and getters, but that's a topic for a another conversation...)*
