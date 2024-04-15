---
layout: post
title:  "AnswerBrawl User Cache"
date:   2024-04-14 21:00:00 -0600
categories: answerbrawl authorization cache
---

{% for category in page.categories %}
  <div style='display: inline; padding: 5px 10px; background-color: #d0d0d0; border-radius: 15px; margin: 5px'>
    {{ category }}
  </div>
{% endfor %}
<br/>

# The Problem
Every call to the db requires authorization. This requires a hit to the db. Because of my (poorly chosen) server locations (api hosted in Can-Central on Azure, db hosted in US-West on Neon), this requires a round-trip with significant latency and adds to the total time of the call. The actual domain code of the method will probably involve at least 1 round-trip to the db, and this can't be helped without moving at least 1 server.

**NOTE**: *I did some ping time tests to various [Azure regions](azurespeed.com) and [AWS regions](https://aws-latency-test.com/) and US West 2 pings seem fastest (taking ~60% of the time).*

# Analysis
Because user data is small, it doesn't change frequently, and is is repeatedly requested, it is a prime target for caching.

API -> Redis would be faster than API -> DB, but will still have latency from the api server to the Redis server.

User data includes:
* Id - ~32 ASCII characters (basically hex-only so can probably be compressed if important)
* Name - ~24 Unicode(?) characters
* Email - ~50 ASCII(?) characters
* Authentication subscription key - ~32 ASCII characters

So probably somewhere around ~200 bytes / user.

At the moment I have 1 user (me!) but I'll say this solution should scale to 1000 users / 100 concurrent before I need to re-evaluate.

# My Solution

I added an in-memory cache that wraps the user repo. The calls to support are:
* Listing calls go straight through to the server... nobody does User listing anyway so not important to cache this.
* Get - need to cache this one
* Get by Auth subscription Key - need to cache this one
* Create - doesn't invalidate anything
* Update - invalidates the Get & GetByAuthKey for the affected user
* Delete - invalidates the Get & GetByAuthKey for the affected user

100 concurrent users *should* take up 200 bytes / user * 100 users = ~20 KB. Plus a little extra for storing the cache keys, cache expiry time, and invalidation id->cache-keys mapping. So this should scale up to many many users and I imagine I'll run into way more problems before this becomes a Memory-Bound bottleneck.
