# nrdocs CMS Concept

## A serverless blogging platform for fresh static content

**nrdocs CMS** is a Git-backed, lifecycle-aware, LLM-assisted blogging platform.

It gives users a simple hosted writing experience while keeping content portable as Markdown and deploying public sites as static assets on Cloudflare R2.

The goal is to combine the ease of a hosted blogging platform with the ownership, auditability, and performance of a static publishing system.

## The short version

nrdocs CMS is designed around a simple idea:

> **Static at request time. Fresh over time. Portable by default.**

A user should be able to create a blog, write posts, schedule publishing, manage content freshness, and deploy globally without maintaining a server, database, or WordPress installation.

Behind the scenes, the system uses Git as the durable source of truth for publishable content. The hosted product can hide that complexity, while still allowing advanced users to export or bring their own repository later.

## What it is

nrdocs CMS is:

- a hosted blogging CMS
- a Markdown-native publishing system
- a static site deployment platform
- a lifecycle-aware content manager
- an LLM-assisted writing and automation layer
- a serverless alternative to traditional WordPress-style hosting

It starts as a blogging platform, but its architecture can later grow into a broader CMS.

## What makes it different

Traditional WordPress sites are powerful because they combine content editing, themes, plugins, and dynamic runtime behavior. But that power comes with operational cost: servers, databases, updates, security hardening, plugin risk, and ongoing maintenance.

Static site generators solve many of those operational problems, but they often create a different problem: content becomes harder to manage and easier to let go stale.

nrdocs CMS aims for a third path:

| Traditional WordPress | nrdocs CMS |
| --- | --- |
| Runtime PHP application | Static output deployed to R2 |
| Database as content source | Git/Markdown as content source |
| Plugins can mutate runtime behavior | Plugins are passive hook subscribers |
| Themes can query application state | Templates receive structured data |
| Manual freshness management | Lifecycle and scheduled freshness workflows |
| Server/database maintenance | Serverless Cloudflare infrastructure |
| Export as an extra feature | Portable project format by default |

## Core principles

The system is built around a few clear boundaries:

1. **Git is the source of truth for publishable content.**  
   Posts, pages, templates, lifecycle rules, generated content, and site configuration are represented as files.

2. **D1 is the control plane.**  
   It stores users, roles, workflow state, deployment records, approval queues, schedules, and audit logs.

3. **R2 stores artifacts.**  
   Media, previews, production builds, source snapshots, and deployment outputs live in object storage.

4. **Workers orchestrate the system.**  
   Workers handle the admin API, scheduling, automation, authentication integration, builds, previews, and deployments.

5. **LLMs are scoped processors.**  
   They receive limited context and return structured suggestions, diagnostics, patches, or artifacts. They do not own state.

6. **Templates are passive renderers.**  
   Templates receive structured site data and produce static output.

7. **Plugins are passive hook subscribers.**  
   Plugins receive defined hook payloads and return structured results. The core validates and applies any output.

8. **Lifecycle is first-class.**  
   Content can define when it should appear, when it becomes stale, when it needs review, and when it should leave the public site.

## Managed and self-hosted

The product follows a model similar to the WordPress.com / WordPress.org split.

### Managed mode

Managed mode is the default experience.

Users do not need a GitHub account, Cloudflare account, R2 bucket, Worker setup, or command-line workflow.

They can:

1. sign up with an email code or identity provider
2. create a blog
3. choose language and direction
4. select a theme
5. write a post
6. publish

Behind the scenes, nrdocs CMS manages the Git repository, Cloudflare resources, deployments, and previews.

### Self-hosted or bring-your-own mode

Advanced users can bring their own repository and Cloudflare account.

This gives them direct control over:

- the Git repository
- content history
- templates
- build workflow
- Cloudflare deployment
- infrastructure policies

Both modes use the same project format, compiler, content model, and lifecycle rules.

## Why start with blogging?

Blogging is a focused starting point with a familiar comparison to WordPress.

A blog needs:

- posts
- pages
- authors
- tags
- categories
- media
- RSS
- sitemap
- templates
- scheduled publishing
- previews
- redirects

That is enough to prove the architecture without trying to build a full general-purpose CMS on day one.

## The promise

nrdocs CMS should feel simple to start with:

> Create a blog, write posts, and publish.

But it should have a deeper technical promise underneath:

> Your content is portable, versioned, lifecycle-aware, and deployed as static output on serverless infrastructure.

## First milestone

The first version should prove that a blog can be:

- easy to create
- pleasant to edit
- deployed statically
- managed without WordPress
- versioned through Git
- localized from day one
- lifecycle-aware
- assisted by AI
- refreshed through scheduled workflows
- exportable as a Markdown project



