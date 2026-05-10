# Why This Exists

## The problem with traditional blogging infrastructure

WordPress became popular because it solved a real problem: it made publishing on the web accessible.

It gave users:

- an admin dashboard
- themes
- plugins
- pages and posts
- media management
- comments
- scheduled publishing
- a large ecosystem

But the traditional WordPress model also carries long-term operational weight.

A typical WordPress site needs:

- a server
- a database
- a runtime application
- updates
- backups
- plugin maintenance
- security hardening
- performance tuning
- cache management

For many blogs and content sites, that infrastructure is heavier than necessary.

## The problem with static sites

Static site generators solve many operational issues.

They are:

- fast
- cheap to host
- secure by default
- easy to cache
- simple to deploy
- friendly to developers

But they often introduce a different set of problems.

Static sites are frequently:

- too technical for non-developers
- hard to edit visually
- disconnected from editorial workflows
- difficult to schedule
- difficult to keep fresh
- lacking a friendly admin UI
- missing lifecycle awareness

A static blog can be technically excellent and still become stale.

## The problem with headless CMSs

Headless CMSs separate content management from presentation, but they often require teams to build or maintain the frontend, deployment flow, preview system, and hosting stack themselves.

That can be powerful, but it adds complexity:

- content lives in one system
- frontend code lives elsewhere
- deployment is another system
- previews require integration work
- migrations can be difficult
- portability varies by vendor

For many blogs, this is more architecture than the user actually wants.

## The problem with AI publishing tools

LLMs can help with writing, editing, summarizing, translating, and reviewing content.

But AI publishing tools can become risky when they are opaque or overly autonomous.

Bad AI publishing systems tend to:

- generate content without clear sources
- mutate content without review
- hide provenance
- encourage low-quality mass generation
- make it unclear what changed and why
- blur the line between human-authored and generated content

nrdocs CMS should use LLMs differently.

The model should be:

> **AI receives scoped context and returns structured suggestions. The system validates them. Humans or policy decide what is applied. Git records the result.**

## The opportunity

There is room for a new kind of blogging platform:

> **A platform that feels as easy as hosted blogging, but behaves like a modern static publishing system.**

The public site should be static and cheap to serve.

The content should be portable and versioned.

The admin should be friendly.

The system should support scheduling, freshness, localization, and AI assistance from the beginning.

## The core thesis

nrdocs CMS is based on this thesis:

> **Most blogs do not need a dynamic runtime database to serve pages, but they do need better tools to manage time, freshness, authorship, and automation.**

Instead of making the public site dynamic, the system makes the publishing pipeline intelligent.

A post can be:

- drafted
- reviewed
- translated
- scheduled
- published
- marked stale
- refreshed
- reviewed again
- unpublished
- archived

The public result is still static output.

## Static at request time, dynamic over time

The key idea is:

> **Static at request time. Dynamic over time.**

A visitor reads static files from the edge.

But behind the scenes, scheduled workflows can:

- publish future posts
- unpublish expired posts
- mark stale posts for review
- generate weekly roundups
- update source-grounded summaries
- refresh metadata
- trigger new builds
- deploy new static output

This gives content the freshness of a dynamic system without requiring every public request to hit a database.

## Lifecycle-aware content

Most CMSs treat content as either draft or published.

nrdocs CMS should understand richer states:

- scheduled
- fresh
- stale
- review due
- unpublished
- archived

Lifecycle metadata lives with the content.

Example:

```yaml
lifecycle:
  publish_at: 2026-07-01T09:00:00+03:00
  fresh_until: 2026-07-08T09:00:00+03:00
  review_at: 2026-07-09T09:00:00+03:00
  unpublish_at: 2026-08-01T09:00:00+03:00
```

Only absolute timestamps are stored. The user interface may offer relative helpers such as “fresh for seven days,” but the CMS resolves those choices into explicit timestamps before writing them to Git.

## Why Git matters

Git is not exposed to every user, but it is important to the architecture.

It provides:

- history
- diffs
- rollback
- review
- portability
- export
- branching
- collaboration
- accountability

This matters even more once LLMs and scheduled automations are involved.

If an automation updates a post, the result should be a Git change.

If an LLM suggests a rewrite, the accepted change should be committed.

If a lifecycle transition unpublishes content, the build should be traceable to a commit and evaluation time.

Git gives the system a durable editorial record.

## Why Cloudflare fits

Cloudflare provides a natural serverless foundation:

- Workers for APIs and orchestration
- D1 for control-plane state
- R2 for media and static deployment artifacts
- CDN for global static delivery
- Access for passwordless/admin authentication

The public site should avoid runtime database reads.

D1 should manage the CMS control plane, not render public content.

R2 should store media and build outputs.

Workers should orchestrate workflows, not become a traditional monolithic server.

## The desired outcome

The user should experience:

> **A simple blogging platform.**

The system should provide:

> **A portable, Git-backed, lifecycle-aware, serverless publishing pipeline.**

The visitor should receive:

> **A fast static site from the edge.**

That combination is the reason this project exists.
