---
layout: post
title:  "Why Not Serverless?"
date:   2019-08-26
tags: snowflake
author: Tim Myerscough
image: img/TODO
---

[Ben Kehoe][ben-kehoe-sls-mindset] recently gave the keynote at [ServerlessDays Sydney][sls-sydney] on the [Serverless Mindset][ben-kehoe-sls-mindset].  He reframes the discussion on serverless, not on the technology, but on the business drivers behind why a Serverless approach makes sense.  This got me to wondering, why shouldn't you use serverless?

Effective use of a Serverless mindset requires learning too much: event-driven architectures, distributed computing and CAP theorem.  By sticking to tried and tested techniques you can avoid having to learn new things.

Computers are eating the world and automation is taking our jobs.  By embracing serverless, you are at risk of becoming redundant.  Embrace "Not Invented Here" to guarantee job security.  Build another queue - you can do it better than everyone else anyway.

- Learning

- Job security

- Security + Networking

- Performance - the dreaded cold start.  Async behaviour.

- Cost - Horror stories (link to lambda recursion).

- Lockin

- Legacy monolith

If you don't want to change; don't want to learn then you shouldn't use serverless.  I can't think of a better reason, can you?

[ben-kehoe-sls-mindset]: TODO
[sls-sydney]: TODO