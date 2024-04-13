---
layout: post
title:  "AnswerBrawl Postgres Migration Update"
date:   2024-04-13 09:00:00 -0600
categories: answerbrawl db postgres
---

{% for category in page.categories %}
  <div style='display: inline; padding: 5px 10px; background-color: #d0d0d0; border-radius: 15px; margin: 5px'>
    {{ category }}
  </div>
{% endfor %}
<br/>

# Finished the Basic Update
Answerbrawl is now running the major entities on Postgres. This includes Questions, Answers, Users, and Tags.

## Encapsulation Wins
Most of the migration went smoothly once the DB schema and wrappers went up. Due to the previous repository encapsulation pattern that I've used for In-Memory, Cosmos, and now Postgres, once I wrapped the repo access on top of prisma, existing <code>await repo.question.get()</code> statements in the domain code worked exactly as before. There were a few required changes, such as changing _etag to etag to conform with some Postgres naming conventions (I think that's what the warning message said), and changing things like question.answerIds to question.answers because I switched to a proper relation table. But overall the standard repo pattern helped.

## Prisma / Postgres Production Debugging
Everything was working fine locally. Once pushed to the "production" Azure server, the api service wouldn't even start up. Github Actions indicated it had been deployed correctly but it gave no response. Between the https://portal.azure.com/#cloudshell/ <code>az webapp log tail</code> and the Portal's Advanced Tools, I found in one of the logs the following error:

<code>
2024-04-13T04:25:18.375673988Z PrismaClientInitializationError: Prisma Client could not locate the Query Engine for runtime "debian-openssl-3.0.x".
2024-04-13T04:25:18.375677588Z 
2024-04-13T04:25:18.375680988Z This happened because Prisma Client was generated for "debian-openssl-1.1.x", but the actual deployment required "debian-openssl-3.0.x".
2024-04-13T04:25:18.375684588Z Add "debian-openssl-3.0.x" to `binaryTargets` in the "schema.prisma" file and run `prisma generate` after saving it:
2024-04-13T04:25:18.375688188Z 
2024-04-13T04:25:18.375691288Z generator client {
2024-04-13T04:25:18.375694388Z   provider      = "prisma-client-js"
2024-04-13T04:25:18.375697688Z   binaryTargets = ["native", "debian-openssl-3.0.x"]
2024-04-13T04:25:18.375701088Z }
</code>

Fantastic error message on Prisma's part. Very clear solution, and everything worked after adding the new binaryTargets.

# Future DB Stuff
I set up the ab frontend and api and cosmos db in Canada East (because I already had some free-tier stuff in Canada Central I believe), at the time thinking I'd stick with Azure for all hosting and could co-locate everything. Now that I'm also using Upstash to host Redis and Neon to host Postgres, that initial location choice is proving to be a thorn in my side.

Options:
1. Use Azure Cache for Redis, Azure Db for PostgreSQL and set up an Azure Virtual Network
2. Relaunch the app & API in an Azure US West location to be closer to my Redis & Postgres.
3. Find other hosting for the app & API co-located with the dbs (I think this would mean rebuilding the SSL certificate)
4. Ignore the problem for now and focus on the 1000's of other things.
