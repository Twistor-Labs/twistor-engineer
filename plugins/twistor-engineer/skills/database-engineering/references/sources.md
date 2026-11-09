# Sources

All entries except S10 were fetched and checked against the primary page on 2026-11-09. S10 (Zepto) was read on 2026-10-30 from the publication's Medium RSS feed, because blog.zepto.com returns 403 to direct fetches (see `research/v1-2-database.md`). Company figures are the company's own claims. Postgres docs were read at version 18.6; the date on those lines is the date read.

- [S1] gh-ost: GitHub's online schema migration tool for MySQL. GitHub, 2016-08-01. https://github.blog/engineering/gh-ost-github-s-online-migration-tool-for-mysql/ (VERIFIED)
- [S2] Online migrations at scale. Stripe, 2017-02-02. https://stripe.com/blog/online-migrations (CORRECTED)
  Note: no exact object count; the post says "hundreds of millions of Subscriptions objects". It does not say the migration was reversible.
- [S3] How Figma's databases team lived to tell the scale. Figma, 2024-03-14. https://www.figma.com/blog/how-figmas-databases-team-lived-to-tell-the-scale/ (VERIFIED)
- [S4] Herding elephants: lessons learned from sharding Postgres at Notion. Notion, 2021-10-06. https://www.notion.com/blog/sharding-postgres-at-notion (CORRECTED)
  Note: the lesson is worded "Shard earlier", not "shard before you need to".
- [S5] Postmortem of database outage of January 31. GitLab, 2017-02-10. https://about.gitlab.com/blog/postmortem-of-database-outage-of-january-31/ (VERIFIED)
- [S6] Postmortem: data loss during database migration. LootLocker, 2026-05-08. https://lootlocker.com/blog/db-postmortem-data-loss-during-database-migration (CORRECTED)
  Note: the source database was deleted about 6 minutes after cutover (10:34 to about 10:40 BST), not 40 minutes as an early summary said.
- [S7] Postmortem: Clerk system outage (March 10, 2026). Clerk, 2026-03-10. https://clerk.com/blog/2026-03-10-service-outage-postmortem (CORRECTED)
  Note: lock contention came from disk latency during a Google Cloud SQL live migration; the remediation is replica promotion, not lock timeouts. The page has no publish date; the date is the incident date.
- [S8] Post Mortem: API Downtime. ImprovMX, 2026-03-06. https://improvmx.com/blog/2026-03-07-post-mortem-api-downtime (VERIFIED)
- [S9] Incident report: October 19, 2026. LLM Gateway, 2026-10-20. https://llmgateway.io/blog/incident-report-october-19-2026 (VERIFIED)
- [S10] Symptoms to Root Cause: Debugging PostgreSQL at Scale. Zepto, 2025-04-15. https://blog.zepto.com/scaling-a-distributed-monolith-our-postgresql-journey-at-zepto-a2da988be62a (VERIFIED)
  Note: read from the full post HTML in the Medium RSS feed (https://blog.zepto.com/feed), not the page itself, which returns 403.
- [S11] PGKeeper: building the bouncer we needed for Postgres. Figma, 2026-05-04. https://www.figma.com/blog/pgkeeper-building-the-bouncer-we-needed-for-postgres/ (VERIFIED)
- [S12] PostgreSQL 18 documentation: client connection defaults (statement_timeout, lock_timeout). PostgreSQL Global Development Group, 2026-11-09. https://www.postgresql.org/docs/18/runtime-config-client.html (VERIFIED)
- [S13] PostgreSQL 18 documentation: ALTER TABLE. PostgreSQL Global Development Group, 2026-11-09. https://www.postgresql.org/docs/18/sql-altertable.html (VERIFIED)
- [S14] PostgreSQL 18 documentation: CREATE INDEX. PostgreSQL Global Development Group, 2026-11-09. https://www.postgresql.org/docs/18/sql-createindex.html (VERIFIED)
- [S15] PostgreSQL 18 documentation: explicit locking. PostgreSQL Global Development Group, 2026-11-09. https://www.postgresql.org/docs/18/explicit-locking.html (VERIFIED)
