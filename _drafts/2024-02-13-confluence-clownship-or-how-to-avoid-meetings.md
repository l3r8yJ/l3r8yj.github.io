---
title: Confluence clownship or How to avoid meetings?
layout: post
date: '2024-02-13'
last_modified_at: '2024-02-13'
categories:
  - management
tags:
  - management
  - codegen
  - docs-as-code
---
How often have you been annoyed by crappy written documentation
in [Confluence](https://www.atlassian.com/software/confluence)?
All these endless wiki pages nested inside each other until your monitor runs out.
Maybe you can remember finding two pages on the same topic that contradict each other?
It is even better when your codebase matches with confluence wiki only 10–20%
of the actual codebase state. 
Or when analytics guys never heard about **formatting** before? 
So, what is the reason for it?

<img width="300" title="Productivity" alt="Productivity" src="/assets/images/making_progress_2x.png">

# Why?
I think the main reason is lack of control.
Comparing with software development,
it's hard to imagine that analysts are cross-reviewing each other,
that they had documentation version control, and in some sense, they don't have any "types" that can
validate their expressions.
> A subtle hint at statically typed languages.

Imagine if they had some sort of CI/CD pipeline that checks their manual documentation work.
Applying some linting and checkstyle to documents.
Life would be much better.

<em/>

# How to be closer to ~~heaven~~ software development?
So, I want to share the experience of our team that successfully avoids this confluence **hell**.
The **First step** toward purification is moving all your documentation 
to git repository as [markdown](https://en.wikipedia.org/wiki/Markdown) files. 
So, if we have a structured format – markdown, it means that we can validate it. 
The **Second step** toward God is to add a CI/CD pipeline with linter and checkstyle to 
beat up on clumsy hands that can't format a piece of text properly.
The **Third step**, gates into Heaven – code review process.
For now, we are close to heaven and developers can control 
everything that comes into knowledge base, looks cool, isn't it? 
Now, when we come to totalitarianism in documentation, 
i.e. a quality gate was built – any documentation manipulations are under our control, and you can
reject any nonsense bullshit in text.

<em/>

# How to avoid any meetings?
The most interesting part of the blog post.
To avoid meetings between programmers and analysts,
we need to force analysts to write documentation which is similar to the code.
It will look native for programmer.
Description of databases is in `sql` files instead of dumb tables in a confluence document.
Description of logic is in `md` files referencing each other in repository.
API specification in `OpenAPI` or `GraphQL` schemas.
Developers just look into the repository and see familiar things,
instead of rereading some paper around 30 times.

<em/>

# Implementation
Unfortunately, it's only relevant for Kotlin or Java developers,
but you can read it, get some ideas and implement them using any tech-stack.
All the tools we're used here are quite popular.
Imagine if your analysts were writing specifications for API, which magically transforms in code,
that you can use.
You might hear about
[API-first](https://blog.dreamfactory.com/api-first-the-advantages-of-an-api-first-approach-to-app-development/)
approach? — We will abuse it at maximum level.
First of all, you need to organize your repository like this:
```asm
contracts-repo/
|
|-- common/
|-- kafka/
|-- microservices/
|–– templates/
|   |
|   |--schema/
|   |   |
|   |   |--schema-name.gql
|   |
|   |-- db/
|   |   |
|   |   |-- table-name.sql
|   |
|   |-- openapi.yaml
|   |-- readme.md
|-- build.gradle.kts
|-- settings.gradle.kts
```
- `common` – directory for common components that can be referenced from any directories
- `microservices` – directory for each microservice, contains folders named like services in your application
- `templates` – template for microservice
  - `schema` – directory for graphql schemas
  - `db` – schema description in `sql`
- `openapi.yaml` – openapi description of your API
- `readme.md` – file for description of anything that happens inside your microservice; has to be named same as microservice.
- `build.gradle.kts`, `settings.gradle.kts` – default files for [Gradle](https://gradle.org/) project.

# How it works?
Each directory inside `microservices`, `kafka`
is publishing as `jar` and all `common` directory is also publishing as single `jar`.
All codegen from openapi and graphql schemas is
generated with several scripts which are described inside of `build.gradle.kts`. 
After generation and publishing, you can add these contracts as dependencies in your microservices.
All you need it just set up a publication inside of `build.gradle.kts` and 
add [gradle-contracts-generator](https://github.com/l3r8yJ/contracts-generator-plugin) plugin.
Plugin will generate `feign` clients,
`DTO`s and API `interfaces` marked with [Spring](https://spring.io/) annotations
and publish them with a specific version. 
The Plugin will be published soon, so stay tuned! 

<em/>

# Pros and Cons
Let's start with Cons.
1. The main "minus" is the analysts in your team,
   if they have any skill issues, it will be challenging to get them to work inside such a process
2. You may forget to update the contracts' dependency version periodically

What about Pros?
1. First of all, there is a single documents' format controlled by CI/CD
2. Ability to review any changes in documentation
3. No extra work – once it's described, once published and used, developers don't need to
   write any boilerplate code
4. Versioning – you can switch versions back and forth
