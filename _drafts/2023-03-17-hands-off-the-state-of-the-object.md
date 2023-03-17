---
title: Hands off the state of the object
layout: post
date: '2023-03-17'
last_modified_at: '2023-03-17'
categories:
- oop
tags: oop setters immutable java maven
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

Let's start with the object-oriented approach.

Take a look at this Java class.
```java

class RequestToApi {
  private URI uri;

  public setUri(URI uri) {
    this.uri = uri;
  }

  public JsonResponse makeResponse() {
    return new JdkRequest(this.uri).response()
  }
}

```

Just typical code example.
```java

RequestToApi req = new RequestToApi();
req.setUri(new URI("https://someawesomeapi.com"))
req.makeResponse();

```
What do we see here? <b>Obviously</b>, this is a violation of the OOP-principle called [encapsulation](https://www.l3r8y.ru/2022/09/03/encapsulation-right-understanding). We literally take an object and change it from the inside, completely changing its behavior! Just imagine if tomorrow someone puts something unsafe in `uri`.


The second thing is that we don't treat our object as a smart object. We just tell it what data to take and what actions to take. 
> "Hey, you have the address to go to and give me the data after you response it!"

We don't treat it as someone smart who can do the work for us. We treat it as an empty shell that has some function for the data we have.


I think more right way to do this looks like this.
```java

class RequestToApi {
  private final URI uri;

  RequestToApi(final URI uri) {
    this.uri = uri;
  }

  public updated(final URI uri) {
    return new RequestToApi(uri);
  }

  public Response response() {
    return new JdkRequest(this.uri).response();
  }
}

```

<br/>

## How to avoid it?

I think the easiest way – is to trust the robots. You don't have to control yourself or anything like that. Just write the code, and when something goes wrong, the robot will just break your arm and tell you that you did something wrong or horrible. That's why I created [this](https://www.l3r8y.ru/sa-tan) guy. So far it only works with [Java](https://en.wikipedia.org/wiki/Java_(programming_language)) projects that use [Maven](https://en.wikipedia.org/wiki/Apache_Maven).

It's just looks into your code and if he sees a mutation of an object, he just fails the build. **He won't even let you build bad code!**

Now that you're fascinated, I'll show you how to work with it.

<br/>

### Diving

All you have to do is add it to your `pom.xml`:
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

<br/>