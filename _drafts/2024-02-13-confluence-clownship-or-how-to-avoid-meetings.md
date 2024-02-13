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

How often you have been annoyed by crappy written documentation
in [Confluence](https://www.atlassian.com/software/confluence)?
All these endless wiki pages nested inside each other until your monitor runs out.
Maybe you can remember finding two pages on the same topic that contradict each other?
It is even better when your codebase matches with confluence wiki only 10–20%
of the actual codebase state. 
Or when analytics guys never heard about **formatting** before? 
So, how can we avoid these issues?

<img width="300" title="Productivity" alt="Productivity" src="/assets/images/making_progress_2x.png">

# Why?
I think the main reason is lack of control.
Comparing with software development,
it's hard to imagine that analysts are cross-reviewing each other,
that they had documentation version control, and in some sense, they haven't any "types" which can
validate their expressions.
> A subtle hint at statically typed languages.

Imagine if they had some sort of CI/CD pipeline that checks their manual documentation work?
Applying some linting and checkstyle to documents.
Life would be much better.

<em/>

# How to be closer to software development?
So, I want to share the experience of our team that successfully avoiding this confluence-hell.
The First step toward purification is moving all your documentation 
to git repository as [markdown](https://en.wikipedia.org/wiki/Markdown) files. 
So, if we have a structured format – markdown, it means that we can validate it. 
You need to add a CI/CD pipeline with linter and checkstyle to 
beat up on clumsy hands that can't format a piece of text properly. The main reason it's not a
CI/CD, it's – **code review process**. For now, developers can control everything that comes into
knowledge base, looks cool, isn't it? Now, when we come to totalitarianism in documentation, 
i.e. a quality gate was built – any documentation manipulations under our control, and you can
reject any nonsense bullshit in text.

<em/>

# How to avoid any meetings?
The most interesting part of the blog post.
To avoid programmer–analyst meetings, we need to force analysts to write documentation which is
similar to the code. 
Description of databases in `sql` files instead of dumb tables in a confluence document. 
It will look more native for programmer. 
Description of logic in `md` files referencing each other in repo, without billions of useless information.
Imagine if your analysts, just writing specifications for API, which magically transforms in code,
that you can use?

<em/>

## Implementation
Suddenly, it's only relevant for Kotlin or Java developers,
but you can read it, get some ideas and implement it using any tech-stack.
All the tools we're used here are quite popular.
You might hear about
[API-first](https://blog.dreamfactory.com/api-first-the-advantages-of-an-api-first-approach-to-app-development/)
approach? — We will abuse it at maximum level.
First of all, you need to organize your repo something like this:
```asm
contracts-repo/
|
|-- common/
|
|-- kafka/
|
|-- microservices/
|
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
|
|-- build.gradle.kts
|-- settings.gradle.kts
```
- `common` – directory for common components that can be referenced from any directories
- `microservices` – directory for each microservice, contains folders named like services in your application
- `templates` – template for microservice
  - `schema` – directory for graphql schemas
  - `db` – schema description in `sql`
- `openapi.yaml` – openapi description of your API
- `readme.md` – file for description anything that happens inside your microservice, have to be named same as microservice.
- `build.gradle.kts`, `settings.gradle.kts` – default files for [Gradle](https://gradle.org/) project.

# How it works?
Each directory inside `microservices`, `kafka` publishing as `jar` and all `common` directory also
publishing as single `jar`. All codegen from openapi and graphql schemas happening with several
scripts which are described inside of `build.gradle.kts`. 
After generation and publishing, you can add these contracts as dependencies in your microservices.
All you need it's just set up a publication inside of `build.gradle.kts` and 
add [gradle-contracts-generator](https://github.com/l3r8yJ/contracts-generator-plugin) plugin.
Plugin will generate `feign` clients, `dto`s and API interfaces marked with [Spring](https://spring.io/) annotations
and publish them with a _specific version_. 
The Plugin will be published soon, so stay tuned! 
