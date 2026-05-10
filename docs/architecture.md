# Architecture

## Overview

nrdocs CMS is built around a clear separation of responsibilities.

The public site is static. The CMS control plane is serverless. The content model is Git-backed. The deployment target is Cloudflare R2.

```text
Author / Automation / LLM
  ↓
CMS Control Plane
  ↓
Git repository
  ↓
nrdocs build
  ↓
R2 deployment
  ↓
Cloudflare CDN
```

The goal is to avoid the traditional WordPress runtime model where every public page request depends on an application server and database.

Instead:

> **The public site is built ahead of time. The CMS manages the publishing pipeline.**

## Responsibility split

### Git

Git is the source of truth for all durable, publishable site material.

This includes:

- posts
- pages
- generated content
- lifecycle metadata
- templates
- theme configuration
- site configuration
- localization files
- redirects
- freshness job definitions
- plugin declarations

If it affects the public site output, it should be represented in Git.

This allows every deployment to be traced back to a versioned project state.

### D1

D1 is the control-plane database.

It stores operational and administrative state, not canonical content.

D1 is responsible for:

- users
- organizations
- sites
- roles and permissions
- Cloudflare Access identity mapping
- Git connections
- workflow runs
- scheduled jobs
- approval requests
- deployment records
- audit logs
- LLM usage records
- build metadata
- editor locks
- temporary autosaves, if needed

D1 should not become a second content management system.

The rule is:

> **D1 may store workflow state. Git stores publishable content.**

### R2

R2 stores large objects and deployment artifacts.

R2 is responsible for:

- uploaded media
- optimized media variants
- preview builds
- production builds
- versioned deployment outputs
- source snapshots
- build logs
- generated reports

The public site can be served from R2-backed static output through Cloudflare.

### Workers

Cloudflare Workers orchestrate the system.

Workers are responsible for:

- admin API
- Cloudflare Access identity validation
- authorization checks
- Git provider API calls
- preview creation
- build orchestration
- deployment activation
- scheduled lifecycle builds
- freshness automations
- webhook handling
- AI provider calls
- routing previews and managed domains

Workers should not turn into a traditional application server for the public site.

The public path should be cache-first and should avoid D1 reads.

### nrdocs compiler

The compiler converts the Git-backed project into static output.

It evaluates:

- Markdown
- frontmatter
- lifecycle metadata
- localization
- templates
- static collections
- plugins or internal hooks
- generated artifacts
- RSS
- sitemap
- search index, if enabled

The compiler should be portable enough that a project can be built outside the hosted platform.

### LLM layer

LLMs are scoped processors.

They receive limited structured context and return structured output.

They can produce:

- diagnostics
- suggested patches
- generated Markdown drafts
- summaries
- metadata suggestions
- translation suggestions
- review reports

They do not directly mutate content, publish sites, access secrets, or own state.

Accepted AI output becomes a Git change.

## Production deployment model

A production deployment should always be explainable as:

```text
Git commit + lifecycle evaluation time + build environment = deployed output
```

A deployment record should include:

- repository reference
- branch
- commit SHA
- build ID
- compiler version
- evaluated_at timestamp
- deployment ID
- active R2 prefix
- build status
- actor or automation that triggered it

Lifecycle makes the evaluation timestamp important. The same commit may produce different output before and after a `publish_at` or `unpublish_at` transition.

The build system should support an explicit `as_of` time for reproducibility.

## Atomic deployments

Build output should be uploaded to a versioned R2 prefix.

Example:

```text
/sites/site_123/deployments/deploy_456/
  index.html
  blog/post.html
  assets/...
```

The active deployment can then be switched by updating a deployment pointer.

This enables:

- atomic activation
- preview deployments
- quick rollback
- deployment history
- safe failed builds

The public site should only switch to a new deployment after the build completes successfully.

## Preview deployments

Previews should use the same build path as production.

Possible preview sources:

- draft branch
- pull request branch
- scheduled future state
- generated post before approval
- lifecycle preview as of a future timestamp

A useful lifecycle-aware feature is:

> **Preview site as of a specific date and time.**

For example:

```text
Preview the blog as it will look on 2026-07-01 at 09:00.
```

This lets editors test scheduled publishing and unpublishing before it happens.

## Content repository abstraction

The CMS should define an internal repository interface.

Example capabilities:

```text
read file
write file
create branch
commit changes
merge branch
list history
generate diff
revert change
create review
```

Implementations can include:

- managed Git repository
- GitHub repository
- GitLab repository later
- local repository for testing

This allows managed and bring-your-own modes to use the same CMS logic.

## Deployment target abstraction

The CMS should also define a deployment target interface.

Example capabilities:

```text
upload build
activate deployment
rollback deployment
create preview URL
list deployments
delete old deployment
```

Implementations can include:

- managed Cloudflare deployment
- user-owned Cloudflare deployment
- static export later

This keeps hosting mode separate from the content model.

## Public request path

The public site should be as cheap and simple as possible.

Ideal public request behavior:

```text
Visitor request
  ↓
Cloudflare cache
  ↓
static asset response
```

If a Worker is used for routing, it should be lightweight and should avoid per-request database work.

Important rule:

> **Public views should not query D1.**

This keeps the hosting model cheap, scalable, and robust.

## Admin request path

The admin path is different.

```text
Editor opens CMS
  ↓
Cloudflare Access authenticates user
  ↓
Worker validates identity
  ↓
Worker checks D1 authorization
  ↓
Worker reads/writes Git, D1, R2, or AI providers as needed
```

Cloudflare Access handles authentication.

The CMS handles authorization.

## Scheduled workflow path

Scheduled workflows are central to the product.

Examples:

- scheduled publishing
- scheduled unpublishing
- stale content review
- RSS-to-draft generation
- weekly roundup generation
- lifecycle transition builds

A typical scheduled workflow:

```text
Cron trigger
  ↓
Worker creates workflow run
  ↓
fetch source data, if needed
  ↓
run LLM or plugin processors, if needed
  ↓
validate output
  ↓
create Git patch or draft branch
  ↓
build preview or production
  ↓
deploy to R2
  ↓
record result in D1
```

For v1, autonomous workflows should create drafts or Git patches unless explicitly allowed to publish.

## Source-of-truth invariant

The most important architectural invariant is:

> **Production deployments are built from Git commits.**

This prevents the system from becoming a mixture of Git content, D1 content, R2 content, and generated runtime state.

D1 may track what happened.

R2 may store what was built.

Git explains what the site is.

## High-level diagram

```text
                    ┌──────────────────────┐
                    │  Cloudflare Access   │
                    └──────────┬───────────┘
                               │
                               ▼
┌─────────────┐      ┌──────────────────────┐
│   Editors   │─────▶│   CMS Worker API     │
└─────────────┘      └──────────┬───────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌─────────────┐        ┌────────────────┐       ┌────────────────┐
│     D1      │        │      Git       │       │      R2        │
│ control     │        │ content/config │       │ media/builds   │
│ plane       │        │ source truth   │       │ artifacts      │
└─────────────┘        └───────┬────────┘       └───────┬────────┘
                               │                        │
                               ▼                        │
                        ┌─────────────┐                 │
                        │ nrdocs build│                 │
                        └──────┬──────┘                 │
                               │                        │
                               ▼                        ▼
                         static output ─────────▶ Cloudflare CDN
```

## Architectural goal

The architecture should make the simple path easy:

> Write, preview, publish.

And the advanced path trustworthy:

> Inspect, diff, export, automate, self-host, and roll back.
