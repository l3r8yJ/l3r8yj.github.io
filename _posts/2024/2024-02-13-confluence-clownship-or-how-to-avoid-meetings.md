---
title: Confluence clownship or How to avoid meetings and DTOs?
layout: post
date: '2024-02-13'
last_modified_at: '2024-02-16'
categories:
  - management
tags:
  - management
  - codegen
  - docs-as-code
---
How often have you been annoyed by crappy written documentation
  in [Confluence](https://www.atlassian.com/software/confluence)?
All these endless wiki pages nested inside each other
  until your monitor runs out.
Do you remember what it's like to find two pages on the same topic that contradict each other;
  or in my experience your codebase matches the Confluence wiki
  only 40–60% of the time?
But much better if analysts never heard about formatting before!
The concrete result: documentation that contradicts the codebase,
  forcing developers into clarification meetings
  to reconcile what the wiki says with what the code does.
So, what is the reason for it?

<img width="300" title="Productivity" alt="Productivity" src="/assets/images/making_progress_2x.png">

# Why?
In my opinion, one main reason is – lack of discipline and control
  (though bad tooling incentives
  and writing for the wrong audience play a role too).
Compared with software development,
  it's hard to imagine that analysts are cross-reviewing each other,
  that they have documentation version control,
  and in some sense,
  they don't have any "types" that can validate their expressions.
> A subtle hint at statically typed languages.

Yes, they have some review and practices.
Usually, though, analysts receive that review from business people —
  not from the tech guys who will develop using the documentation.
Imagine if they had a CI/CD pipeline that checks their artifacts,
  applying linting and checkstyle.
Life would be much better.

<em/>

# How to take control of documentation?
So, I want to share the experience of our team
  that successfully avoids this confluence hell.
This worked in our context: a team with technically literate analysts,
  a microservice architecture, and a Spring-based JVM stack.
Your results depend on similar conditions.
The First step toward purification is –
  moving all your documentation to git repository as [markdown](https://en.wikipedia.org/wiki/Markdown) files.
So, if we have a structured format – markdown, it means we can validate it.
The Second step toward God is –
  to add a CI/CD pipeline with linter and checkstyle
  to beat up on clumsy hands that can't format a piece of text properly.
The Third step, gates into a Heaven – code review process.
For now, we are close to heaven.
Developers can control everything that comes into the knowledge base —
  looks cool, isn't it?
All issues that were unclear are now resolved during the review process,
  rather than at the time of development,
  thereby reducing the chance of a clarification meeting about requirements.
Now, when we come to totalitarianism in documentation,
  i.e. a quality gate was built –
  any documentation manipulations are under our control,
  and you can reject any nonsense bullshit in text.

<em/>

# Why does control help us to dodge meetings?
The most interesting part of the blog post.
To avoid clarification meetings between programmers and analysts
  about requirements,
  we need to force analysts to write documentation
  which is similar to the code.
This works best when analysts already have some technical background;
  teams with non-technical analysts
  will need to invest time in tooling training first.
It looks native for the programmer.
It reduces the number of misunderstandings between developer and analyst.
Description of databases is in [SQL](https://en.wikipedia.org/wiki/SQL) scripts,
  logic is in Markdown format,
  API specifications in [OpenAPI](https://swagger.io/specification/)
  or [GraphQL](https://graphql.org/) schemas.
Developers look into the repository and see familiar things,
  instead of struggling with unstructured Confluence content.
The next sections show how this approach helps avoid writing boilerplate code.

<em/>

# Boilerplate dodging
Unfortunately we use [Spring](https://spring.io/),
  so the tech part is only relevant for [JVM](https://en.wikipedia.org/wiki/Java_virtual_machine) developers,
  but you can read it, get ideas, and implement them using any tech-stack.
All the tools we use here are popular.
Imagine if your analysts were writing specifications for API,
  which magically transform into code you can use.
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
- `common` – directory for common components
  that can be referenced from any directories of contracts-repo
- `microservices` – directory for each microservice,
  contains folders named like services in your application
- `templates` – template for microservice
  - `schema` – directory for `.graphql` schemas
  - `db` – directory for schemas' descriptions in `.sql` scripts
- `openapi.yaml` – openapi description of your API
- `readme.md` – file for description of anything that happens inside your microservice;
  has to be named same as microservice.
- `build.gradle.kts`, `settings.gradle.kts` –
  default files for [Gradle](https://gradle.org/) project.

# How it works?
OpenAPI and GraphQL schemas generate the boilerplate code.
The build publishes all directories inside `microservices` and `kafka`
  as separate `JAR`s.
The `common` directory is also published as a separate `JAR`.
What exactly does it produce?
1. `DTO`s for each declared type
2. API `interfaces`, marked with Swagger and Spring annotations, like
   `@Controller`, `@PostMapping`, and `@RequestBody`
3. [Feign clients](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/)
   for outer clients, that will communicate with our service

After generation and publishing,
  you can add these contracts as dependencies in your microservices.
All you need to do is set up a publication inside `build.gradle.kts`
  and add [gradle-contracts-generator](https://github.com/l3r8yJ/contracts-generator-plugin) plugin.
_Note: the plugin is still in development and not yet published at time of writing.
Until it is available, the same result can be achieved with existing stable tools:
[openapi-generator](https://github.com/OpenAPITools/openapi-generator) for OpenAPI and
[graphql-codegen](https://the-guild.dev/graphql/codegen) for GraphQL schemas.
Put a star on the repo if you want the Gradle plugin published faster!_

<em/>

# Pros and Cons
Let's start with Cons
1. The main "disadvantage" is the analysts in your team,
   if they have any skill issues, it will be challenging to get them to work inside such a process
2. You may forget to update the contracts' dependency version periodically
3. Analysts must stay 1–2 sprints ahead of developers —
   contracts must exist before developers can generate code from them.
   If analysts fall behind, developers block.

What about Pros?
1. A single document format controlled by CI/CD
2. Ability to review any changes in documentation –
   all changes coming from [pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests).
   This gives all the benefits of [git](https://git-scm.com/).
3. Opportunity to write documentation in modern IDEs or code editors,
   like [Writerside](https://www.jetbrains.com/writerside/?utm_source=product&utm_medium=link&utm_campaign=TBA),
   [IntelliJ IDEA](https://www.jetbrains.com/idea/),
   or [VS Code](https://code.visualstudio.com/)
4. All migrations are defined in one place
5. No extra work – once it's described, once published and used,
   developers don't need to write any `DTO` boilerplate
6. Versioning – you can switch versions of your API back and forth

In short: fewer clarification meetings
  because structured, reviewable contracts force ambiguities
  to be resolved in text before development starts.
No handwritten DTOs because the contracts generate them.
