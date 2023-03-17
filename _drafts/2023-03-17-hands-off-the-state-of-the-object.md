---
title: Hands off the state of the object
layout: post
date: '2023-03-17'
last_modified_at: '2023-03-17'
categories:
- oop
tags: oop setters immutable
---

How often do you see the creation of an object through configuration? What does that mean? Right, we have a mutable state of the object. Of course, this does not happen without the involvement of setters.

What's wrong with setters, you may ask. Well, absolutely **anything**! Enough of the fact that, for the human brain, the alterability of things is a very unnatural thing. Yes, I know that there are no things in our world that are completely immutable. But it's a bit of a tricky thing for our brains.

<br/>

For guys who need proof:

  - Piaget, J. (1952). The origins of intelligence in children. New York: International Universities Press.

  - Carey, S. (2009). The origin of concepts. Oxford: Oxford University Press.

  - Leslie, A. M. (1987). Pretense and representation: The origins of “theory of mind.” Psychological Review, 94(4), 412–426.

  - Scholl, B. J., & Tremoulet, P. D. (2000). Perceptual causality and animacy. Trends in Cognitive Sciences, 4(8), 299–309.


<br/>


Now that we have dealt with the physiological side of the question, let's move on to the practical.

<br/>

## Why it's completely unusable?

<br/>

<br/>

## How to avoid it?

I think the easiest way – is to trust the robots. You don't have to control yourself or anything like that. Just write the code, and when something goes wrong, the robot will just break your arm and tell you that you did something wrong or horrible. That's why I created [this](https://www.l3r8y.ru/sa-tan) guy. So far it only works with [Java](https://en.wikipedia.org/wiki/Java_(programming_language)) projects that use [Maven](https://en.wikipedia.org/wiki/Apache_Maven).

This guy just looks into your code and if he sees a mutation of an object, he just fails the build. **He won't even let you build bad code!**

Now that you're fascinated, I'll show you how to work with it.

<br/>

### Diving

First of all you need to add it to your `pom.xml`:
```xml
<build>
  <plugins>
    <plugin>
      <groupId>ru.l3r8y</groupId>
      <artifactId>sa-tan</artifactId>
      <!-- version might be outdatet -->
      <version>0.1.1</version>
      <executions>
        <execution>
          <goals>
            <goal>search</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

In fact, that's it, we're ready to use.

<br/>