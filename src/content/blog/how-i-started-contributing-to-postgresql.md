---
title: "How I Started Contributing to PostgreSQL"
description: "A look into my journey starting open-source database development, from submitting my first pgexporter patch to building progress tracking for pgmoneta."
pubDate: 2026-04-23
updatedDate: 2026-04-25
tags: ["PostgreSQL", "Contribution", "Getting-started"]
image:
  src: "/images/postgresql-journey.png"
  alt: "PostgreSQL Contribution Journey"
---

A.A

## Motivation

I'm writing this because I know how scary open source can feel - especially when you’re staring at a huge codebase or a low-level project like a database. It’s easy to think, “this isn’t for me.”

But I want to show that you don’t need to be a “database expert :D” to get started - you just need curiosity and the courage to try.

In this article, I'll walk through the work I've done (not just the technical parts, but the growth behind them), and highlight a few lessons that really stuck with me along the way.

Enjoy reading !

---
## Introduction

At the beginning of 2026, I decided to try contributing to an open-source database project. At that point, my experience was pretty basic - mostly relational theory from university - so I wasn't sure what to expect.

I had heard of PostgreSQL as a large and well-known database, but I didn't really understand how big its community and ecosystem were until I started exploring it myself. PostgreSQL isn't just a database engine - it's a whole [ecosystem](https://wiki.postgresql.org/wiki/Ecosystem:PostgreSQL_ecosystem) of tools for monitoring, backups, performance, connection pooling, and more.

While exploring, I came across [Mnemosyne Systems](https://www.mnemosyne-systems.ai/opensource/why/), founded by Jesper Pedersen. What drew me in was the combination of clean C codebases, clear documentation, and active mentorship - exactly what I needed.

Three projects stood out to me, each solving a different real-world problem:

* [`pgexporter`](https://github.com/pgexporter/pgexporter): Collects PostgreSQL metrics and exposes them via Prometheus for monitoring.
* [`pgmoneta`](https://github.com/pgmoneta/pgmoneta): A backup and restore solution for PostgreSQL.
* [`pgagroal`](https://github.com/pgagroal/pgagroal): A high-performance connection pool for PostgreSQL.

---
## A simple patch is often just a doorway to challenges you didn't know you were ready for

My journey started with `pgexporter`. It’s written in C, a language I hadn’t used before, but since I write a lot of C++, the difference didn’t feel too big. It only took me a couple of days of reading the codebase to get comfortable with the syntax.

I started with a "Good First Issue" ([#384](https://github.com/pgexporter/pgexporter/pull/384)), where I built, installed, and configured the tool as an end user, then made a small fix. Because the [pgexporter docs](https://pgexporter.github.io/pgexporter.html) are clear and simple, it was a smooth first step, and the community was supportive whenever I got stuck.

That first merged PR gave me that **adrenaline dose** to go deeper. I picked up a more challenging issue ([#388](https://github.com/pgexporter/pgexporter/pull/388)), where I implemented Prometheus support using an **[Adaptive Radix Tree (ART)](https://ieeexplore.ieee.org/document/6544812)** for fast lookups.

This part really caught my attention, especially since I enjoy algorithms and data structures. It was a great chance to learn a new indexing structure and also get introduced to [Prometheus](https://prometheus.io) and observability in general for a first time.

Next, I worked on issue ([#394](https://github.com/pgexporter/pgexporter/pull/394)) to improve how time-based configurations are handled. Many duration-related attributes (e.g. `metrics_cache_max_age`, `metrics_query_timeout`, etc) were stored as plain integers, which made the code harder to manage. This was the first issue where I had to make a real design decision, since C doesn’t have a built-in duration type for converting between units like seconds and minutes.

So I looked at how other languages handle this. Go was a helpful reference, especially functions like `Duration.Minutes()` and `Duration.Milliseconds()`. I ended up implementing something similar - a `Duration` type in C.

Since these projects follow a similar design, the mentors suggested applying the same change to **project sisters** (pgmoneta, pgagroal). It was my first time doing that. Even though they are separate projects, they share a lot of infrastructure and coding style, so once you get used to one, moving to another feels natural.

> You don’t need to start with big issues. Even small ones help you understand the codebase and tools, and they naturally lead you to bigger and more meaningful contributions.

---
## Don't wait for a door - build one

At that point, I truly felt like I was part of the community, and I wanted to help the tool grow more, so I asked my mentor what to do next. He said: “Look at other monitoring solutions, figure out what’s missing, and implement it.” That was a great lesson—the community is really open to new ideas and improvements.

I started doing some research and worked on new metrics like long-running transactions ([#403](https://github.com/pgexporter/pgexporter/pull/403)), Multixact ID Age (`datminmxid`) ([#407](https://github.com/pgexporter/pgexporter/pull/407)), and `pg_stat_walreceiver` ([#409](https://github.com/pgexporter/pgexporter/pull/409)). This helped me understand PostgreSQL views like `pg_database`, `pg_catalog`, and `pg_stat_statements`, and how `pgexporter` exposes metrics.

I also got my first experience with [Grafana](https://grafana.com), so I watched some demos and read the docs to understand how everything connects. I noticed that we needed a centralized way to handle Grafana dashboards (pg13–pg18), so I discussed an idea for a Python script that automatically fetches the version ([#441](https://github.com/pgexporter/pgexporter/issues/441)). This idea came from seeing a similar approach used with the metrics `.yaml` file.

At the same time, I kept reading the codebase and understanding more, and I started asking myself what could be improved. I also asked my mentor whenever something was unclear. One thing I noticed was duplicated cache logic between `prometheus.c` and `bridge.c`, so I opened an issue ([#422](https://github.com/pgexporter/pgexporter/issues/422)). The idea was welcomed, and I started working on a centralized cache in `cache.c` so both Prometheus and Bridge could use it. It also makes it easier to extend later if new components need caching.

> You don’t always need to wait for existing issues. Sometimes just reading the codebase, noticing gaps, and starting a discussion is enough. You can create your own opportunities by suggesting improvements and opening new ideas.

---

## Small bugs can hide bigger problems

Reporting bugs is actually one of the things I learned the most from. No code is ever 100% bug-free. Bugs are normal. But catching one is always valuable, because it makes the project better and improves the experience for users.

So whenever you notice something strange - even if you're not sure it's a bug - it's worth asking a mentor. From my experience, while reading docs and using the tools, I found things like broken links or features that didn’t work. Most of them weren’t critical (e.g. [#429](https://github.com/pgexporter/pgexporter/issues/429), [#438](https://github.com/pgexporter/pgexporter/issues/438), [#411](https://github.com/pgexporter/pgexporter/issues/411)), and some turned out to be invalid issues (e.g. [#401](https://github.com/pgexporter/pgexporter/issues/401)). But small improvements still matter. And sometimes, you end up finding something bigger.

That’s exactly what happened to me one day. I was using the tool heavily when it suddenly crashed. The logs showed a null pointer reference. The quick fix would have been to just add a null check - but I paused and asked myself: _is this really the right fix?_ I wasn’t sure.

So I opened an issue [#491](https://github.com/pgexporter/pgexporter/issues/491) and discussed it with my mentor. It turned out the problem was deeper than I expected. Instead of patching it, we decided to trace the root cause. I started adding logs and debugging step by step, sharing everything along the way.

Eventually, we discovered that Prometheus would crash at a critical point when some attribute values were `null`. From there, we discussed a better solution. Since the PostgreSQL protocol also provides data types (e.g. int, bool, etc.), we decided to use that information and store them. Whenever a null value is detected, we replace it with a default value based on its type.

In the end, we fixed it [(#492)](https://github.com/pgexporter/pgexporter/pull/492). What started as a simple null pointer issue turned into improving part of the Prometheus infrastructure itself.

> Always report bugs when you find something suspicious, and take time to understand the root cause instead of rushing to fix it. What looks small at first can sometimes lead to something more important.

---

## I Built an Alert Rules System

As I spent more time exploring the system, I wanted to push myself further by taking on a more complex challenge—something that would require more design thinking. That’s when I noticed a gap: there was no built-in alert rules system.

pgexporter already exposes many metrics, but there wasn’t an easy way to define alerts out of the box or customize them in a simple way.

### 1. The problem

We needed a way to define alerts without hardcoding them, and it had to be flexible enough for different use cases (override any attribute, define their own alerts).

The idea was to support a simple YAML-based configuration, where users can define their own alerts or override existing ones:

```yaml
alerts:
  # Override built-in connection alert target servers
  - name: postgresql_down
    servers: primary

  # Override built-in connections_high threshold to 90%
  - name: connections_high
    threshold: 90

  # Custom new query alert
  - name: replica_lag_high
    description: Replication lag exceeds 500MB
    type: query
    query: "SELECT pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) FROM pg_stat_replication"
    operator: ">"
    threshold: 524288000
    servers: [replica]
```

So the goal was clear:
* keep it lightweight
* avoid hardcoding
* support multiple clusters (primary/replica)

It also needed to integrate well with tools like Grafana (annotations) and support notifications via webhooks (e.g., Slack).
### 2. Start small

Since this is a large feature, my mentor suggested starting with the manual. At first, I didn’t get why, but it helped me think from the user’s perspective - how the feature should be used and how it fits with other tools.

Then we built a simple initial version, focusing only on correctness. Optimization came later.

### 3. Making it lightweight

After the initial patch, I needed to measure the performance of the system. I used [ab - Apache HTTP server benchmarking tool](https://httpd.apache.org/docs/2.4/programs/ab.html). At first, the system was quite slow - the overhead was around ~1.1s on each Prometheus scrape, which is too high.

So I started analyzing the code more closely, going through it line by line until I found the bottleneck:

```c
for (int a = 0; a < config->number_of_alerts; a++)
  for (server = 0; server < config->number_of_servers; server++)
    bool conn_valid = pgexporter_connection_isvalid(..) // sends SELECT 1
```

The problem here is that `SELECT 1` requires opening a new connection and running a query. So the complexity becomes:

* O(number_of_alerts × M)
* where M is the time for a single `SELECT 1`

At that point, I started thinking about how to remove the dependency on `number_of_alerts`. This is where my algorithms background helped. I remembered the idea of dynamic programming "memoization".
Instead of running `SELECT 1` multiple times for the same server, I cached the result and reused it in later iterations. (check this part [here](https://github.com/pgexporter/pgexporter/blob/94622504e2788070798380d9d6c4b0bd3462ed6a/src/libpgexporter/prometheus.c#L1625-L1628))

This small change reduced the overhead from ~1.1s to ~0.01s. 

### 4. Make it easy to configure
Another challenge was the design of how configuration should work.

The flow is simple:
1. Read each alert from the YAML file
2. Check if the alert already exists (override), and keep the rest of the attributes unchanged

The tricky part was step 1. I needed a way to know which attributes were actually provided in the YAML, so later I can decide what to override and what to keep as default.

My first idea was to add a flag for each attribute, but that didn’t feel right - it would make the code messy and hard to extend.

So again… I tried to use some algorithms thinking xD

Instead of flags, I used a bitmask. Each attribute has a bit, and when parsing the YAML, I set the bit if that attribute exists. Later, I just check the mask to know what should be overridden.

This made the design much cleaner, and adding new attributes became straightforward - just assign a new bit. (check this part [here](https://github.com/pgexporter/pgexporter/blob/94622504e2788070798380d9d6c4b0bd3462ed6a/src/libpgexporter/alert_configuration.c#L265-L291)).

### 5. Final result

In the end, the work was done ([#414](https://github.com/pgexporter/pgexporter/pull/414)) after about one and a half months of continuous work, improvements, and testing.

> Building software takes time. You don’t need to get everything right at the beginning - start simple, and improve it step by step. Take your time with the design; there’s no need to rush.

---

## I Built Progress Support for Database Backup

Once I got familiar with pgexporter, I wanted to push myself further by trying a different project and taking on a bigger challenge, so I started looking into pgmoneta issues.

### 1. The problem

The task was to add progress support for base backup - basically, showing how much of the backup is done while it’s running. At first, this might sound simple, but it’s not that straightforward.

To show progress, you need two things:
* how much work is done
* how much work there is in total
### 2. Read PostgreSQL Code

To understand that, I started reading PostgreSQL’s [basebackup_copy.c](https://github.com/postgres/postgres/blob/77c7a17a6e5fefcd55edb6b47fc462a059b983dc/src/backend/backup/basebackup_copy.c#L250-L251). It was my first time going that deep into the PostgreSQL engine.

I found that PostgreSQL already solves this problem. Before starting the backup stream, it walks through the data directory and tablespaces, calculates the total size of all files, and sends that number first. Then, during the backup, it keeps track of how many bytes are sent.

So in the end, we have:
* total bytes
* processed (done) bytes

With these two values, we can estimate progress. I also used a simple idea to estimate the remaining time using: `done_bytes / elapsed_time` → gives a rough speed

Based on this, I implemented progress tracking for base backup ([#946](https://github.com/pgmoneta/pgmoneta/pull/957)).
### 3. When things get more complex

After that, my mentor asked me to extend this beyond base backup to cover all phases (manifest, SHA512, compression, encryption, etc.), and also support restore and verify.

This is where things got more interesting.

Unlike base backup, the full process is not a single step -  it’s multiple stages, and each stage runs after the previous one. That means when the process starts, we don’t really know how long the future stages will take.

So the question became: **how do you show progress of something that you don’t know how long it will take?**

I looked at how other tools handle this and used [Jenkins](https://www.jenkins.io) as a reference.

The idea I proposed was simple:
* give each stage an estimated weight (how much it contributes to total time)
* then adjust these weights later based on real execution time

To estimate the initial weights, I used [pgbench](https://www.postgresql.org/docs/current/pgbench.html) and ran benchmarks with different datasets to measure how long each stage takes.

### 4. Final result

In the end, I built a reusable `progress.c` so it can also support restore and verify later. After some discussion and refinement, it was merged in ([#1097](https://github.com/pgmoneta/pgmoneta/pull/1097)).

### 5. Again... A small discovery that led to more

While running benchmarks for progress support, I wrote a small script for it. During those runs, I noticed some backup operations were being rejected, so I reported it ([#1013](https://github.com/pgmoneta/pgmoneta/issues/1013)). It turned out to be a race condition, which led me to work on a new finalize workflow (currently in progress: [#1052](https://github.com/pgmoneta/pgmoneta/pull/1052)). 

> When you don’t know how to solve something, look at how others solved it - you don’t need to start from zero.

---

## Conclusion

Contributing to database projects doesn't require deep expertise - just the basics and learning as you go. Even my first patches had nothing directly to do with the database itself.

What's next for me? I'm diving deeper into `pgagroal` - understanding connection pooling at the protocol level. There's a lot to build, and I'm excited to keep going.

If any of this made you curious, don't just read about it - pick a project, find a "Good First Issue", and submit your first patch. What made the biggest difference for me was the community - great mentors, real mentorship, and people who want to help you grow. [Mnemosyne Systems](https://www.mnemosyne-systems.ai/opensource/why/) is where I found that.

Finally, I would like to thank my mentors Jesper Pedersen and Luca Ferrari - all of this wouldn’t have been possible without them.

Thanks for reading !