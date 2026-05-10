# Content Lifecycle

## Beyond draft and published

Most CMSs treat content as either draft or published.

nrdocs CMS should treat content as something with time semantics.

A post may be:

- scheduled for the future
- fresh for a limited time
- due for review
- stale but still visible
- unpublished after a campaign ends
- archived for historical reference

This is especially important for blogs, news posts, product announcements, event posts, temporary campaigns, generated posts, and evergreen content that must be reviewed over time.

## Core idea

A content lifecycle defines when content should appear, when it becomes stale, when it needs review, and when it should leave the public site.

Lifecycle metadata lives with the content in Git.

Example:

```yaml
---
title: "Election Guide"
slug: /blog/election-guide

lifecycle:
  publish_at: 2026-06-01T09:00:00+03:00
  fresh_until: 2026-07-01T23:59:59+03:00
  review_at: 2026-07-02T09:00:00+03:00
  unpublish_at: 2026-07-08T00:00:00+03:00
---
```

The build system evaluates this metadata and determines how the content behaves.

## Absolute timestamps only

The stored lifecycle model should use absolute timestamps only.

The CMS user interface may offer helpful relative controls:

- publish tomorrow morning
- keep fresh for seven days
- unpublish after the campaign ends
- review one day after the event

But the CMS should resolve those choices into explicit timestamps before writing to Git.

Good:

```yaml
lifecycle:
  fresh_until: 2026-07-08T09:00:00+03:00
```

Avoid:

```yaml
lifecycle:
  fresh_for: 7d
```

This removes ambiguity.

Questions like “seven days from what?” should be answered by the CMS at the moment the user configures the lifecycle.

## User-facing time vs technical time

There are two layers.

### User-facing time

The user may choose dates in a friendly way:

```text
Publish next Monday at 9:00
Review one week after publication
Unpublish after the event
```

### Technical time

The repository stores resolved timestamps:

```yaml
lifecycle:
  publish_at: 2026-07-01T09:00:00+03:00
  review_at: 2026-07-08T09:00:00+03:00
```

The compiler and scheduler only evaluate technical time.

This keeps builds predictable and avoids hidden relative-date rules.

## Required timezone clarity

All lifecycle timestamps should be timezone-aware.

Preferred:

```yaml
publish_at: 2026-07-01T09:00:00+03:00
```

Also acceptable:

```yaml
publish_at: 2026-07-01T06:00:00Z
```

Avoid storing timezone-less dates.

The site configuration should define a default timezone for user-facing input:

```yaml
site:
  timezone: Asia/Jerusalem
```

If a user enters a local time in the CMS, the CMS resolves it into a timestamp with an explicit offset.

## Lifecycle fields

The v1 lifecycle model should remain small.

### `publish_at`

When the content becomes eligible for public output.

Before this time, the content is hidden from:

- public pages
- navigation
- tag pages
- category pages
- RSS
- sitemap
- search index

### `fresh_until`

When the content stops being considered fresh.

After this time, the content may still be visible, but the CMS can mark it as stale.

Stale content can trigger:

- dashboard warnings
- review tasks
- AI freshness review
- template banners
- lower search/recommendation priority
- scheduled refresh workflows

### `review_at`

When the content should be reviewed.

This is an editorial signal.

After this time, the content remains visible unless other lifecycle fields say otherwise.

The CMS should show the item as review due.

### `unpublish_at`

When the content should be removed from public output.

Unpublished content should be excluded from:

- public pages
- navigation
- indexes
- RSS
- sitemap
- search

Important:

> **Unpublish does not mean delete from Git.**

The file remains in the repository and can be restored, updated, archived, or republished later.

## Derived lifecycle states

The build system can derive lifecycle states from timestamps.

Suggested states:

```text
scheduled
active_fresh
active_stale
review_due
unpublished
```

Example evaluation:

```text
now < publish_at
  → scheduled

publish_at <= now <= fresh_until
  → active_fresh

now > fresh_until
  → active_stale

now > review_at
  → review_due

now > unpublish_at
  → unpublished
```

The exact state model can be refined, but the system should distinguish stale content from unpublished content.

## Default v1 behavior

Suggested defaults:

| State | Public page | Nav | RSS | Sitemap | Search |
| --- | --- | --- | --- | --- | --- |
| Scheduled | No | No | No | No | No |
| Active fresh | Yes | Yes | Yes | Yes | Yes |
| Active stale | Yes | Yes | Yes | Yes | Yes |
| Review due | Yes | Yes | Yes | Yes | Yes |
| Unpublished | No | No | No | No | No |

Templates may optionally show banners for stale or review-due content.

## Lifecycle is evaluated at build time

The public site remains static.

Lifecycle is not evaluated on every request.

Instead:

```text
build starts
  ↓
compiler reads content from Git
  ↓
compiler evaluates lifecycle metadata as of build time
  ↓
compiler includes, excludes, or annotates content
  ↓
static output is deployed
```

This means lifecycle transitions require builds.

If a post should publish at 09:00, the control plane should schedule a build at or shortly after 09:00.

## Lifecycle-driven scheduled builds

Lifecycle metadata should feed the scheduler.

If a repository contains content with:

```yaml
publish_at: 2026-07-01T09:00:00+03:00
unpublish_at: 2026-07-08T00:00:00+03:00
```

the control plane can derive required future builds:

```text
2026-07-01 09:00
  build site to publish content

2026-07-08 00:00
  build site to unpublish content
```

Multiple lifecycle transitions should be batched into a single build when possible.

## Reproducibility

Because lifecycle depends on time, a deployment is defined by more than a commit.

A precise deployment record should include:

```text
Git commit
evaluated_at timestamp
compiler version
plugin/template versions
build environment
```

This allows the system to reproduce the deployed output.

The compiler should support an explicit evaluation time:

```text
nrdocs build --as-of 2026-07-01T09:00:00+03:00
```

## Lifecycle and previews

Lifecycle-aware preview is a major feature.

Editors should be able to preview:

- the current draft
- a scheduled post before it is public
- the full site as it will look at a future time
- the site after an unpublish transition
- stale banners before they appear

Example:

```text
Preview this blog as of 2026-07-01 at 09:00.
```

This helps editors verify time-based behavior before it goes live.

## Lifecycle and generated content

Generated content should usually have lifecycle metadata.

Example:

```yaml
---
title: "Daily Infrastructure Brief — 2026-05-08"
generated: true
generated_by: daily-infrastructure-brief
published_at: 2026-05-08T07:00:00+03:00

lifecycle:
  publish_at: 2026-05-08T07:00:00+03:00
  fresh_until: 2026-05-09T07:00:00+03:00
  review_at: 2026-05-09T08:00:00+03:00
  unpublish_at: 2026-06-08T07:00:00+03:00
---
```

This prevents generated posts from accumulating forever without review.

## Lifecycle and cost control

Lifecycle can also control operational cost.

For example, after `unpublish_at`, a post can be excluded from:

- public output
- RSS
- sitemap
- search indexes
- public AI retrieval
- scheduled refresh jobs
- recommendation systems

This is useful for:

- temporary campaigns
- news posts
- daily briefings
- event posts
- limited-time pages
- stale generated content

The goal is not only to reduce infrastructure cost, but also to reduce editorial debt.

## Lifecycle and localization

Lifecycle should be per locale.

An English post may be published today, while a Hebrew translation may be scheduled for tomorrow.

Example:

```yaml
locale: he
translation_key: why-static-blogs-go-stale

lifecycle:
  publish_at: 2026-05-09T09:00:00+03:00
```

Each translation is its own content file with its own lifecycle metadata.

This gives editors precise control over multilingual publishing.

## Lifecycle and AI

Lifecycle events can trigger AI-assisted workflows.

Examples:

- after `fresh_until`, review this post for stale claims
- after an event date, suggest converting future-oriented language into past tense
- before `review_at`, summarize what should be checked
- before `unpublish_at`, suggest a redirect or archive note
- after source updates, generate a proposed refresh

The AI should not mutate content directly.

It should return structured diagnostics or patches.

## Lifecycle and redirects

When content is unpublished, the default behavior should be 404 or no output.

Later, the lifecycle model can support actions:

```yaml
lifecycle:
  unpublish_at: 2026-08-01T09:00:00+03:00
  on_unpublish:
    action: redirect
    to: /archive/election-2026
```

For v1, redirects can be handled separately through a Git-backed redirects file.

## What v1 should include

The first version should include:

- `publish_at`
- `fresh_until`
- `review_at`
- `unpublish_at`
- absolute timestamps only
- lifecycle evaluation during build
- scheduled lifecycle builds
- lifecycle state in admin UI
- lifecycle-aware previews
- no automatic deletion from Git

## What v1 can postpone

The first version can postpone:

- complex lifecycle action rules
- event DSLs
- conditional lifecycle logic
- automatic redirect generation
- per-plugin lifecycle overrides
- runtime lifecycle evaluation
- automatic Git file deletion

## Principle

The lifecycle model should be simple, explicit, and Git-backed.

> **Content should know when it appears, when it becomes stale, when it needs review, and when it leaves the public site.**
