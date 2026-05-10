# Managed vs Self-Hosted

## The WordPress.com / WordPress.org analogy

nrdocs CMS should follow a familiar two-mode model:

```text
Managed nrdocs
  like WordPress.com

Self-hosted / bring-your-own nrdocs
  like WordPress.org
```

The difference is that both modes should use the same underlying project format.

A managed user can start without knowing anything about Git or Cloudflare, while an advanced user can take full control of the repository and infrastructure.

## Why this split matters

The core architecture uses Git and Cloudflare.

That is powerful, but it can create onboarding friction.

A user should not need to do this before publishing their first post:

```text
create GitHub account
create repository
install GitHub app
create Cloudflare account
create R2 bucket
configure Workers
connect domain
configure Access
```

That is too much setup for a blogging platform.

The default experience should be:

```text
sign up
create blog
choose language
choose theme
write post
publish
```

The system can still be Git-backed internally.

The user does not need to manage the Git repository themselves.

## Managed mode

Managed mode is the default product experience.

In managed mode, nrdocs provides:

- hosted admin UI
- passwordless or identity-provider login
- managed Git repository
- managed Cloudflare deployment
- managed R2 storage
- managed previews
- managed scheduled jobs
- managed AI integration
- nrdocs subdomain
- optional custom domain support

The user sees a blogging platform.

The platform manages the publishing system.

## Managed mode user experience

A new user should be able to onboard with a simple flow:

1. Enter blog name
2. Choose default locale and text direction
3. Pick a starter theme
4. Choose a subdomain
5. Start writing
6. Publish

Behind the scenes, the platform provisions:

- site record in D1
- managed Git project
- starter content scaffold
- theme configuration
- R2 deployment prefix
- first static build
- preview and public URLs

The user should not be asked to connect GitHub or Cloudflare unless they choose an advanced path.

## Managed Git

Managed mode should still use Git internally.

The platform can create and manage a private repository or Git-compatible content store for the site.

The user-facing UI should abstract Git concepts.

| Git concept | User-facing concept |
| --- | --- |
| Commit | Version |
| Branch | Draft |
| Merge | Publish |
| Pull request | Review |
| SHA | Version ID |
| Revert | Restore |
| Conflict | Edit conflict |

This keeps the benefits of Git without exposing unnecessary complexity.

## Managed Cloudflare

Managed mode should also hide Cloudflare setup.

The platform owns and operates the Cloudflare resources needed to host the site.

A managed user gets:

```text
https://example.nrdocs.site
```

They can later connect a custom domain.

Advanced users can later connect their own Cloudflare account or export the site.

## Self-hosted or bring-your-own mode

Self-hosted mode is for users who want direct ownership and control.

They can bring:

- their own GitHub or GitLab repository
- their own Cloudflare account
- their own R2 bucket
- their own Worker deployment
- their own domain and Access policies
- their own CI/build workflow

This mode should appeal to:

- developers
- agencies
- technical publishers
- enterprises
- compliance-sensitive teams
- users who want direct infrastructure control

## Same project format

The managed and self-hosted modes should share the same project structure.

A project should be portable.

Example:

```text
nrdocs.yml
content/
  posts/
  pages/
  authors/
theme/
  tokens.yml
  overrides/
.nrdocs/
  freshness.yml
  redirects.yml
```

The hosted CMS may manage this project behind the scenes, but the files should remain meaningful and exportable.

## Export as a first-class feature

Managed mode should not mean lock-in.

A user should be able to export:

- Markdown project ZIP
- full Git repository history
- media manifest
- templates and theme config
- lifecycle metadata
- redirects
- freshness job config

Possible export paths:

```text
Download project ZIP
Download Git bundle
Push to GitHub
Transfer managed repository
Move to self-hosted deployment
```

The promise should be:

> **Start managed. Leave with your content and history anytime.**

## Progressive ownership

The product can support a gradual ownership path.

### Level 1: Fully managed

```text
managed Git
managed Cloudflare
managed subdomain
hosted CMS
```

Best for users who just want to write and publish.

### Level 2: User-owned content repository

```text
user connects GitHub
platform still hosts deployment
CMS writes to user repo
```

Best for developers and teams that want content ownership but not infrastructure management.

### Level 3: User-owned repository and infrastructure

```text
user owns Git
user owns Cloudflare
CMS can still provide UI/workflows
```

Best for advanced teams and enterprises.

### Level 4: Fully self-hosted

```text
user runs the compiler and deployment workflow independently
hosted CMS is optional
```

Best for users who want maximum control.

## What must remain portable

To make this model honest, managed-only hidden state must be avoided.

Anything that affects public output should exist in the project files.

Portable project state includes:

- posts
- pages
- authors
- tags and categories
- lifecycle metadata
- localization metadata
- templates
- theme tokens
- redirects
- freshness job definitions
- plugin declarations
- generated content

Non-portable operational state can live in D1:

- user sessions
- permissions
- workflow run history
- deployment logs
- approval queues
- AI usage records
- temporary editor locks

This keeps the project exportable without requiring the operational history to be part of the public site.

## Friction reduction

The managed model removes the biggest adoption blockers:

```text
No GitHub required.
No Cloudflare required.
No command line required.
No server required.
No database setup required.
No password database required.
```

The user can start with the simple product and discover the underlying technical power later.

## Developer trust

The self-hosted path gives advanced users trust.

They can inspect:

- content files
- lifecycle metadata
- Git history
- templates
- deployment outputs
- generated content
- AI-applied patches

They can also move away from the hosted service without losing their site.

This makes the managed product easier to trust.

## Product positioning

For non-technical users:

> A simple serverless blogging platform that keeps your posts fresh.

For technical users:

> A Git-backed static publishing system with lifecycle-aware automation.

For teams and enterprises:

> A portable Cloudflare-native CMS that can run on managed or customer-owned infrastructure.

## Key principle

The managed product should hide complexity, not remove ownership.

The self-hosted product should expose control, not require a different content model.

Both should be powered by the same underlying system.
