---
layout: post
title:  "AnswerBrawl: Changing Cosmos -> Postgres"
date:   2024-04-03 13:00:00 -0600
categories: answerbrawl db cosmos postgres
---

### Relational vs Document Db
[AnswerBrawl](https://answerbrawl.com)'s data is highly relational, and so although Cosmos NoSQL has been really quick to try out for free and get me rolling, it's not appropriate for the main entities when I want to do some more filtering & joins.

#### Basic highly relational Entities:
* Questions n-n Answers (e.g. a Question must have many potential Answers, and an Answer may be the answer for more than 1 Question)
* Questions n-n Tags (e.g. a Question should have more than one Tag, and a Tag should refer to multiple Questions)
* Answers n-n Tags (e.g. an Answer should have more than one Tag, and a Tag should refer to multiple Answers)
* Users n-n Questions (e.g. Favourite Questions, Owned Questions)

#### Other "Entities":
Ballots: short-lived and referenced only by Id, so continuing to use Redis for these is still appropriate.

### Hosting
With [PlanetScale](https://planetscale.com) cancelling the free tier for MySQL, I looked around to find some alternatives. I'm not opposed to paying for Db hosting, but until (if) AnswerBrawl
pays for itself in some way, I want to keep it as cheap as possible. $40 / month.

[Neon](https://neon.tech) provides free-tier Postgres hosting. Since my knowledge of SQL doesn't extend to the pros & cons of MySQL vs Postgres vs SQL Server, I'm happy to try out Postgres.

