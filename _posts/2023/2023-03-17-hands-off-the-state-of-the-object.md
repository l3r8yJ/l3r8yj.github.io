---
title: Hands off the state of the object
layout: post
date: '2023-03-17'
last_modified_at: '2023-03-19'
categories:
- oop
tags: oop setters immutable java maven
---

<img height="500" title="TMNT in Java" alt="TMNT in Java" src="/assets/images/mutants.png">

How often do you see the creation of an object through configuration? What does that mean? Right, we have a mutable state of the object. Of course, this does not happen without the involvement of setters.
What's wrong with setters, you may ask. Well, absolutely **everything**! The fact that, for the human brain, the mutability of objects is a very unnatural thing is enough to prove it. Yes, I know that there are no things in our world that are completely immutable. But it's a bit of a tricky thing for our brains.
For guys who need proof:
  - Piaget, J. (1952). The origins of intelligence in children. New York: International Universities Press.
  - Carey, S. (2009). The origin of concepts. Oxford: Oxford University Press.
  - Leslie, A. M. (1987). Pretense and representation: The origins of “theory of mind.” Psychological Review, 94(4), 412–426.
  - Scholl, B. J., & Tremoulet, P. D. (2000). Perceptual causality and animacy. Trends in Cognitive Sciences, 4(8), 299–309.
Now that we have dealt with the physiological side of the question, let's move on to the practical one.

## Why is it completely unusable?

Take a look at this Java class.
```asm

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

Just a typical code example.
```asm

RequestToApi req = new RequestToApi();
req.setUri(new URI("https://someawesomeapi.com"))
req.makeResponse();
// some code
req.setUri(new URI("https://otherawesomapi.com"))
req.makeResponse();

```
What do we see here? <b>Obviously</b>, this is a violation of the OOP-principle called [encapsulation](https://www.l3r8y.ru/2022/09/03/encapsulation-right-understanding).
We literally take an object and change it from the inside, completely changing its behavior! Just imagine if tomorrow someone puts something unsafe in `uri`.
The second thing is that we don't treat our object as a smart object. We just tell it what data to take and what actions to take. 
> "Hey, you have the address to go to and give me the data after you response it!"

We don't treat it as someone smart that can do the work for us. We treat it as an empty shell that has some function for the data we have.
I think that proper and object-oriented way to do this is the following one.
```asm

class RequestToApi {
  private final URI uri;

  RequestToApi(final URI uri) {
    this.uri = uri;
  }

  public RequestToApi updated(final URI uri) {
    return new RequestToApi(uri);
  }

  public JsonResponse response() {
    return new JsonResponse(
      new JdkRequest(this.uri).response()
    );
  }
}

```
The use of this class is much better now.
```asm

final RequestToApi req = new RequestToApi(
  new URI("https://someawesomeapi.com")
);
req.updated(new URI("https://otherawesomapi.com")).response(); 

```
Here, on the other hand, we set the immutable essence of an object at "birth", saying this after.

  > "Please give me a response from the server."

Now we treat the object as someone smart to rely on. In case we need to change the server, we simply create a new object through `updated()`. This way, those who worked with the object before won't be affected by the change, because it's immutable. As a nice bonus, such a class will be **thread-safe**!
As a small summary, I see a lot of pluses in this approach, which seem to me more important than the disadvantages. You may also want to check out the books written on the subject. The first one is "[Object Thinking](http://davewest.us/product/object-thinking/)" by David West and the second one is "[Elegant Objects](https://www.yegor256.com/elegant-objects.html)", v1-v2, by Yegor Bugayenko.



## How to avoid setters?

I think the easiest way is to trust the robots.
You don't have to control yourself or anything like that.
Write the code,
and when something goes wrong,
the robot will just break your arm and tell you that you did something wrong or horrible.
That's why I created [this](https://www.l3r8y.ru/sa-tan) guy.
So far,
it only works with [Java](https://en.wikipedia.org/wiki/Java_(programming_language)) projects.
Is just looks into your code, and if it sees a mutation of an object, it just fails the build.
It won't even let you build bad code!**
Now that you're fascinated, I'll show you how to work with it.

### Diving

All you have to do is to add it to your `pom.xml`:

```

<build>
  <plugins>
    <plugin>
      <groupId>ru.l3r8y</groupId>
      <artifactId>sa-tan</artifactId>
      <!-- version might be outdated -->
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

Now I'll give you some examples:

```asm

class IAmGoodBoy {
  private final String name;

  IAmGoodBoy(final String name) {
    this.name = name;
  }
}

```

As you can see this class `IAmGoodBoy` is good, there is no reason to send a warning here.

```asm

class IAmBadBoy {
  private String name;

  public void rename(String name) {
    this.name = name;
  }
}

```

But this guy `IAmBadBoy` – a real devil in the flesh. That's why `Sa-tan` will say this.
```bash

.../IAmBadBoy.java': Method 'IAmBadBoy#rename' has wrong method signature,
because method body contains an assignment, setters violates OOP principles

```

But if life has forced you to have bad classes in your code, you can annotate them.

```asm

@Mutable
class IAmBadBoy {
  private String name;

  public void rename(String name) {
    this.name = name;
  }
}

```

Now `Sa-tan` won't say anything about it!



*Thank you; I hope this post was interesting for you, also you can correct me in the comments if I made mistakes, etc.*
