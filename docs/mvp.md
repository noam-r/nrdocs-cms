# MVP Scope

## Goal

The first version should prove one focused idea:

> **A blog can be easy to publish, serverless to host, Git-backed underneath, lifecycle-aware, localized from day one, and assisted by AI without becoming opaque or unsafe.**

The MVP should not try to become a full WordPress replacement immediately.

It should be a sharp blogging product with a clear path toward a broader CMS later.

## Product definition

The MVP is:

> **A managed serverless blogging platform with Git-backed Markdown content, lifecycle-aware publishing, AI-assisted editing, and static deployment to Cloudflare R2.**

The user should experience a simple hosted blogging platform.

The system should maintain a portable Git/Markdown project behind the scenes.

## Target users

The initial target users are:

- technical founders
- developer advocates
- small product teams
- independent writers who want ownership
- agencies building lightweight blogs
- teams that dislike WordPress maintenance
- publishers experimenting with AI-assisted editorial workflows

The first version should not target every possible WordPress user.

It should be especially appealing to people who care about:

- performance
- low maintenance
- content ownership
- static deployment
- Markdown portability
- scheduled publishing
- multilingual content
- AI-assisted editing

## Primary promise

The MVP should communicate:

> **Start writing like a hosted blog. Keep the ownership of a Git-backed static site.**

A user should not need to create a GitHub account or Cloudflare account to start.

But the project should always be exportable.

## Core user flow

A first-time user should be able to:

1. Sign up with email code or identity provider
2. Create a blog
3. Choose default locale and text direction
4. Choose a starter theme
5. Write a post
6. Preview the post
7. Publish the blog
8. Share a public URL

Behind the scenes, the platform:

1. Creates a managed site record
2. Creates or provisions a managed Git-backed project
3. Commits starter files
4. Builds the site
5. Uploads output to R2
6. Activates a public deployment
7. Records deployment metadata

## MVP features

### Managed blog creation

The user can create a blog without external accounts.

The platform provides:

- hosted admin UI
- managed content repository
- managed deployment
- managed subdomain
- default theme
- default locale

### Markdown post editor

The editor should support:

- title
- body
- excerpt
- SEO description
- tags
- category
- cover image
- lifecycle fields
- preview

The editor can be WYSIWYG, Markdown-first, or split-view, but the stored content should be Markdown.

### Posts and pages

The MVP should support:

- blog posts
- static pages
- tags
- categories
- authors, at least minimally

Suggested structure:

```text
content/
  posts/
  pages/
  authors/
```

### Frontmatter

Each post should store metadata in frontmatter.

Example:

```yaml
---
title: "Why Static Blogs Go Stale"
slug: /blog/why-static-blogs-go-stale
locale: en
dir: ltr
description: "Static blogs are fast, but keeping them fresh is hard."
excerpt: "A lifecycle-aware publishing model can keep static blogs fresh over time."
author: default
tags: [cms, static-sites, publishing]
category: publishing
cover_image: null

created_at: 2026-05-08T09:00:00+03:00
updated_at: 2026-05-08T09:30:00+03:00
published_at: 2026-05-08T09:30:00+03:00

lifecycle:
  publish_at: 2026-05-08T09:30:00+03:00
  fresh_until: 2026-06-08T09:30:00+03:00
  review_at: 2026-06-09T09:30:00+03:00
  unpublish_at: null

ai:
  assisted: false
  last_reviewed_at: null
---
```

### Lifecycle-aware publishing

The MVP should include:

- `publish_at`
- `fresh_until`
- `review_at`
- `unpublish_at`
- lifecycle state in the CMS
- scheduled lifecycle builds
- lifecycle-aware previews
- no deletion from Git

Default behavior:

| State | Behavior |
| --- | --- |
| Scheduled | Hidden from public output |
| Fresh | Visible normally |
| Stale | Visible, marked in admin |
| Review due | Visible, highlighted for review |
| Unpublished | Removed from public output |

### Preview builds

Users should be able to preview:

- draft posts
- scheduled posts
- the full blog before publishing
- lifecycle behavior as of a future date

The preview system should use the same build path as production.

### Production deploy to R2

The MVP should deploy static output to R2.

Deployments should be versioned.

A deployment record should include:

- commit or version ID
- evaluated_at timestamp
- build status
- deployment ID
- active R2 prefix
- public URL
- actor or workflow that triggered it

### Rollback

Users should be able to restore a previous deployment.

Rollback can initially mean:

> Reactivate a previous R2 deployment output.

It does not need to create a Git revert in v1, although that can be added later.

### Localization and RTL foundation

The MVP should support multiple locales from the beginning.

At minimum:

- site default locale
- locale-aware content files
- RTL/LTR theme support
- locale-aware date formatting
- locale-aware URLs
- one official RTL-compatible theme

A first version can support a single active locale per site if necessary, but the content and theme model should not block multiple locales.

### Security

The MVP should avoid custom passwords.

It should use:

- Cloudflare Access or equivalent identity layer
- magic-code or identity-provider login
- D1-based authorization
- simple roles: owner, editor, viewer
- audit logs for sensitive actions

The CMS should not store user passwords.

### AI assistance

The MVP should include focused AI actions:

- generate title alternatives
- generate excerpt
- generate SEO description
- suggest tags
- rewrite selected text
- review post clarity
- review stale content
- summarize deployment changes

AI output should require user approval before changing content.

### Freshness automation

The MVP should include one safe automation:

> **RSS/source links to draft post.**

Flow:

```text
user configures source URLs
  ↓
automation fetches source items
  ↓
LLM summarizes supplied sources
  ↓
CMS creates a draft post
  ↓
editor reviews and approves
  ↓
post is committed and deployed
```

This proves the fresh-static concept without fully autonomous news publishing.

### Export

Managed users should be able to export their site.

Minimum export:

- Markdown content
- frontmatter
- templates or theme config
- lifecycle metadata
- localization metadata
- redirects
- media manifest

Preferred export:

- project ZIP
- Git history bundle or push-to-GitHub option

## Admin sections

The MVP admin UI can be organized as:

```text
Posts
Pages
Media
Freshness
Deployments
Design
Settings
```

### Posts

Shows:

- drafts
- scheduled
- published
- stale
- review due
- unpublished

### Freshness

Shows:

- lifecycle calendar
- stale posts
- review-due posts
- scheduled automations
- generated drafts
- failed workflow runs

### Deployments

Shows:

- preview builds
- production builds
- deployment history
- active deployment
- rollback actions
- build logs

### Design

Shows:

- selected theme
- design tokens
- logo
- colors
- typography
- RTL preview, if applicable

### Settings

Shows:

- site name
- locales
- domain
- export
- users
- roles
- AI usage
- automation limits

## Out of scope for MVP

The first version should not include:

- full plugin marketplace
- full theme marketplace
- arbitrary custom content types
- comments
- memberships
- e-commerce
- forms
- public AI chatbot
- runtime personalization
- plugin-owned D1
- D1-only content mode
- full WordPress importer
- complex enterprise approval workflows
- multi-tenant agency dashboards
- advanced visual page builder
- high-frequency live data widgets
- open-web autonomous news publishing

These can be added later if the core product works.

## Internal architecture in MVP

Even if third-party plugins are not public yet, the MVP should use internal hook points.

Internal hooks can support:

- lifecycle evaluation
- RSS generation
- sitemap generation
- SEO metadata validation
- AI review
- search index generation
- deployment summaries

This prepares the system for future extensibility without launching a public ecosystem too soon.

## MVP success criteria

The MVP is successful if users can say:

```text
I created a blog without setting up servers.
I published a fast static site.
I can schedule posts.
I can see which posts are stale.
AI helps me write and review posts.
My content is exportable.
I do not need WordPress.
```

Technical success criteria:

```text
Production builds come from Git-backed content.
Public traffic does not require D1.
Lifecycle transitions trigger builds.
AI outputs are structured and auditable.
Deployments are versioned and reversible.
The project can be exported.
RTL/localization assumptions are built in.
```

## First release positioning

The first release should be positioned as:

> **A lifecycle-aware, AI-assisted serverless blogging platform.**

Not yet:

> A complete WordPress replacement.

The larger CMS vision can remain visible, but the MVP should be focused enough to build and explain.
