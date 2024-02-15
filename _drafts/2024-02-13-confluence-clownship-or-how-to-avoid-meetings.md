---
title: Confluence clownship or How to avoid meetings and DTOs?
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
When your codebase matches with confluence wiki, only 10–20%
of the actual codebase state.
But much better if analysts never heard about formatting before!
So, what is the reason for it?

<img width="300" title="Productivity" alt="Productivity" src="/assets/images/making_progress_2x.png">

# Why?
In my opinion, the main reason is – lack of control.
Comparing with software development,
it's hard to imagine that analysts are cross-reviewing each other,
that they had documentation version control, and in some sense, they don't have any "types" that can
validate their expressions.
> A subtle hint at statically typed languages.

Yes, they had some review and practices, but mainly it's happening from "business" people,
not from tech guys, not from the people who are going to develop using this documentation.
Imagine if they had some sort of CI/CD pipeline that checks their documentation,
applying some linting and checkstyle.
Life would be much better.

<em/>

# How to take control of documentation?
So, I want to share the experience of our team that successfully avoids this confluence hell...
The First step toward purification is – moving all your documentation 
to git repository as [markdown](https://en.wikipedia.org/wiki/Markdown) files. 
So, if we have a structured format – markdown, it means that we can validate it. 
The Second step toward God is – to add a CI/CD pipeline with linter and checkstyle to 
beat up on clumsy hands that can't format a piece of text properly.
The Third step, gates into a Heaven – code review process.
For now, we are close to heaven and developers can control 
everything that comes into knowledge base, looks cool, isn't it? 
Now, when we come to totalitarianism in documentation, 
i.e. a quality gate was built – any documentation manipulations are under our control, and you can
reject any nonsense bullshit in text.

<em/>

# How to dodge meetings & DTOs?
The most interesting part of the blog post.
To avoid meetings between programmers and analysts,
we need to force analysts to write documentation which is similar to the code.
It will look native for programmer
and will reduce the number of misunderstandings between developer and analyst.
Description of databases is in [SQL](https://en.wikipedia.org/wiki/SQL) scripts,
logic is in Markdown format, API specifications in [OpenAPI](https://swagger.io/specification/)
or [GraphQL](https://graphql.org/) schemas. 
Developers just look into the repository and see familiar things,
instead of struggling with wiki-pages, tables, etc. In the next sections we will see how this
approach can help us to avoid writing boilerplate code... 

<em/>

# Implementation
Unfortunately we use [Spring](https://spring.io/),
that's why the tech part is
only relevant for [JVM](https://en.wikipedia.org/wiki/Java_virtual_machine) developers,
but you can read it, get some ideas and implement them using any tech-stack.
All the tools we're used here are quite popular.
Imagine if your analysts were writing specifications for API, which magically transforms in code,
that you can use.
You might hear about
[API-first](https://blog.dreamfactory.com/api-first-the-advantages-of-an-api-first-approach-to-app-development/)
approach?
— We will abuse it at maximum level.
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
  of contracts-repo
- `microservices` – directory for each microservice, contains folders named like services in your application
- `templates` – template for microservice
  - `schema` – directory for `.graphql` schemas
  - `db` – directory for schemas' descriptions in `.sql` scripts
- `openapi.yaml` – openapi description of your API
- `readme.md` – file for description of anything that happens inside your microservice; has to be named same as microservice.
- `build.gradle.kts`, `settings.gradle.kts` – default files for [Gradle](https://gradle.org/) project.

# How it works?
Boilerplate code from OpenAPI and GraphQL schemas will be generated.
All directories inside `microservices`, `kafka` will be published as separate `JAR`s.
The `common` directory also would be published as separate `JAR`.
What exactly will be produced?
1. `DTO`s according to types that where declared
2. API `interfaces`, marked with Swagger and Spring annotations, like
   `@Controller`, `@PostMapping`, `@RequestBody`, etc.
3. [Feign clients](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/)
   for outer clients, that will send requests to our service

After generation and publishing, you can add these contracts as dependencies in your microservices.
Everything that you need is just set up a publication inside of `build.gradle.kts` and 
add [gradle-contracts-generator](https://github.com/l3r8yJ/contracts-generator-plugin) plugin.
Plugin will publish them with a specific version. 
_The Plugin will be published soon, so stay tuned!_

<em/>

# Pros and Cons
Let's start with Cons.
1. The main "minus" is the analysts in your team,
   if they have any skill issues, it will be challenging to get them to work inside such a process
2. You may forget to update the contracts' dependency version periodically
3. Analysts should go further than developers in 1–2 sprints. 

What about Pros?
1. First of all, there is a single documents' format controlled by CI/CD
2. Ability to review any changes in documentation – all changes coming from 
   [pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)
3. Opportunity to write documentation in modern IDEs or code editors, 
   like [Writerside](https://www.jetbrains.com/writerside/?utm_source=product&utm_medium=link&utm_campaign=TBA),
   [IntelliJ IDEA](https://www.jetbrains.com/idea/),
   [VS Code](https://code.visualstudio.com/), etc. 
4. All migrations will be defined in one place 
5. No extra work – once it's described, once published and used, developers don't need to
   write any boilerplate code as `DTO`
6. Versioning – you can switch versions of your API back and forth
