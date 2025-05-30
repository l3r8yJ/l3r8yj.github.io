---
title: Don't be shy – cry!
layout: post
date: '2023-08-06'
last_modified_at: '2023-08-06'
categories:
  - opensource
  - report
  - coding
tags:
  - opensource
  - bugs
  - reporting
---

Please tell me, how often do you see someone's code that doesn’t make your stomach, back, or eyes hurt? I’m sure this is not uncommon, especially in projects with legacy code. So how can we reduce the number of such situations? In my opinion, several things can be done.

<img weight="520" title="Crying guy" alt="Crying guy" src="/assets/images/crying-guy.jpeg">

## Go cry!
You need to find the person who wrote this work of art and the person who approved it –
very often these people are in your team.
Then you go to GitHub, GitLab, Jira etc. and create a ticket for them,
where you’re asking them why this code is written this way.
If they ignore you, go and cry to your boss with words like,
“These guys are interfering with my work.”
Tell him how unprofessionally this work was done, it is impossible to maintain!
Say everything you think about this code.



## Don't upset people
You have to understand these things;
all the code that you write at work will be read by someone, will be maintained by someone.
Tests which you write will fail on someone’s code change, 
and when this guy or girl tries to find reason why tests failed, they won't have
to spend 2–3 hours to make clear why you’re comparing dto1 and dto2 in test called `compareTest`.
Also tell yourself, never, never do this.
```asm
@Test
void givenNone_whenGetAuthorizedPersons_andNothing_andFoundEmpty_thenOk() {
    this.authorizedPersonController.getAllPersons(
        new AuthorizedPersonFilterDto(),
        Sort.unsorted()
    );
    assertTrue(true);
}
```
I was out of my mind when I discovered this piece of crap.
The guy had just written code with an `assertTrue(true)` statement to keep the coverage level.
But even more questions and righteous anger were directed to the reviewer who said:

> _"LGTM, merge."_

I can’t imagine how irresponsible one would have to be to leave something like that.



## Don't cry twice

After you find something you can’t live with in this project,
you should make your team add linter, checkstyle config, etc. to the project.
Explain why no one should fall victim to your laziness, irresponsibility and slackness.



## Don't be lazy

Try to spend more time thinking about the design.
Don't be in a hurry, keep a cool head to avoid such disappointments.
```asm
public int calculateMatchedCount(
            KmbAsyncRequest request,
            KmbRule rule,
            Set<DivisionLimitV2> limits,
            String regionCode,
            BigDecimal checkSum
    ) {
        int count = 0;
        if (!filterByGszLimits(checkSum, List.of(rule), limits).isEmpty()) {
            count++;
        }
        if (!filterByPersonLimits(checkSum, List.of(rule), limits).isEmpty()) {
            count++;
        }
        if (validRegion(regionCode, rule)) {
            count++;
        }
        if (validClientClass(request, rule)) {
            count++;
        }
        if (validBehaviorRateLevel(request, rule)) {
            count++;
        }
        if (validDealType(request, rule)) {
            count++;
        }
        return count;
    }
```
How are we supposed to maintain that?



## Make tools more annoying

What if you don't have the right quality control tool –
create one or add functionality to an existing one!
This is a real example of how I found a bad code in my work project. 
I forced the team to use [this](https://github.com/volodya-lombrozo/jtcop/) plugin
and then contributed this tool to make my project impossible to build with this code.
The Algorithm is simple:
1. Create an [issue.](https://github.com/volodya-lombrozo/jtcop/issues/242)
2. Send a [pull request.](https://github.com/volodya-lombrozo/jtcop/pull/249)
3. Update tool to a new version with your contribution.
4. Fix issues in your project to make it work again.

These advices may seem pretty obvious to you, 
and all this thing may sound like
_"We are against everything bad and for everything good."_
But I make
you **do** things instead of **knowing** about the problem
and nodding your head with a clever look.

Thanks!