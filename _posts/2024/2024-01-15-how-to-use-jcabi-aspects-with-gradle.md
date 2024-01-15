---
title: How to use jcabi-aspects with gradle
layout: post
date: '2024-01-15'
last_modified_at: '2024-01-15'
categories:
  - guide
tags:
  - jcabi-aspects
  - gradle
  - aop
  - aspectj
  - kotlin
  - jooq
---
This is a short guide on how to make [jcabi-aspects](https://aspects.jcabi.com/) work with [gradle](https://gradle.org/) projects. This approach will work with `Java`, `Kotlin`, `Scala` and `Groovy`.
<img height="420" title="AOP Weaving" alt="AOP Weaving" src="/assets/images/weaving.png">
First of all, add `jcabi-aspects` dependency into your `build.gradle.kts` file
```asm
dependencies {
    implementation("com.jcabi:jcabi-aspects:${latestVersion}")
}
```
To make jcabi's annotations work, we have to [weave](https://eclipse.dev/aspectj/doc/next/devguide/ltw.html#weaving-class-files-more-than-once) our classes with aspects from jcabi. To do this, we need a [plugin](https://docs.gradle.org/current/userguide/plugins.html) which will modify our compiled sources. Best option is – [freefair post-compile weaving plugin](https://docs.freefair.io/gradle-plugins/8.4/reference/#_post_compile_weaving).
```asm
plugins {
    id("io.freefair.aspectj.post-compile-weaving") version "${latestVersion}"
}
```
After that, you should mark jcabi dependency as a `aspect` in `dependencies`:
```asm
dependencies {
    implementation("com.jcabi:jcabi-aspects:${latestVersion}")
    aspect("com.jcabi:jcabi-aspects:${latestVersion}")
}
```
For most cases it's should be enough. But if you have some plugins, which have their own `compile` tasks, you can face a problem like
```asm
Execution failed for task ':service-name:someCompileTask'.
> Cannot infer AspectJ class path because no AspectJ Jar was found on class path
```

All you need to do its to expand your compile task with your classpath that contain `aspectj.jar` in your `build.gradle.kts`:
```asm
tasks.someCompileTask {
    configure<AjcAction> {
        classpath.setFrom(configurations.compileClasspath)
    }
}
```

Full version of `build.gradle.kts` is:
```asm
plugins {
    id("io.freefair.aspectj.post-compile-weaving") version latestVersion
}

dependencies {
    implementation("com.jcabi:jcabi-aspects:${latestVersion}")
    aspect("com.jcabi:jcabi-aspects:${latestVersion}")
}

// Only if you need
tasks.someCompileTask {
    configure<AjcAction> {
        classpath.setFrom(configurations.compileClasspath)
    }
}
```
Thanks!