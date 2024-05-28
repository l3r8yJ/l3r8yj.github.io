---
title: Testing role model with Spring Boot
layout: post
date: '2024-05-24'
last_modified_at: '2024-05-24'
categories:
  - spring-boot
tags:
  - testing
  - spring-boot
  - spring-security
---

Recently, our team encountered a problem. We need to release a feature that
uses a role model with a lot of business logic depending on different roles.
The implementation was ready, integration tests were written, and services were deployed in a test
environment.
So it's time to throw new features to QA engineers...

<img width="300" title="How software development works" alt="How software development works" src="https://imgs.xkcd.com/comics/software_development.png">

The first idea that came to our mind was to simply create multiple accounts for each role and use
them.
But we ran into a problem.
For some reason, our company has simply restricted the use of test accounts.
By the way, we can't use swagger functions for testing because of GraphQL.
We need a solution that can be used with different tools, different platforms, etc.
because the tester has to test in an environment that he is used to.
<br/>

## Spring Boot starer

Here I'm introducing a super-simple spring-boot starter that allows us to test the role model
via `http` header.

[spring-x-roles-authorities-starter](https://github.com/l3r8yJ/spring-x-roles-authorities-starter)
allows
you to simply add the `X-Roles` header to your request, and after authorization, the roles that you
specified will be set for your request.

```asm
// Basic authentication
GET https://example.com/endpoint
Authorization: // any auth
X-Roles: my-role, another-role
```

Allows hitting this endpoint, even if your real account doesn't contain this role:

```asm
@RestController
final class Controller {

    @GetMapping("/endpoint")
    @PreAuthorize("hasAnyAuthority('my-role')")
    public ResponseEntity<String> sayHi() {
        return "Hi!";
    }
}
```

That's all, thanks!
