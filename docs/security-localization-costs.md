# Security, Localization & Costs

## These are v1 requirements

Security, localization, and cost modeling should not be treated as later concerns.

They affect the architecture from the beginning.

The platform should be:

- multilingual from day one
- RTL-compatible from day one
- passwordless-first from day one
- Cloudflare-native from day one
- cost-aware from day one

## Localization from day one

Localization should be structural, not an afterthought.

The platform should support:

- multiple locales
- locale-aware URLs
- locale-specific posts and pages
- locale-aware lifecycle metadata
- locale-aware SEO metadata
- locale-aware RSS feeds
- locale-aware sitemap entries
- RTL and LTR rendering
- locale-aware date formatting
- AI-assisted translation and review

## Recommended content structure

A good structure groups translations under one logical content item.

Example:

```text
content/
  posts/
    why-static-blogs-go-stale/
      en.md
      he.md
      ar.md
```

Each locale file has its own frontmatter.

Example:

```yaml
---
title: "למה בלוגים סטטיים מתיישנים"
locale: he
dir: rtl
translation_key: why-static-blogs-go-stale
slug: /he/blog/why-static-blogs-go-stale

lifecycle:
  publish_at: 2026-05-09T09:00:00+03:00
  fresh_until: 2026-06-09T09:00:00+03:00
  review_at: 2026-06-10T09:00:00+03:00
  unpublish_at: null
---
```

Lifecycle is per locale because translations may be published, reviewed, or unpublished at different times.

## Locale configuration

The site config should define supported locales.

Example:

```yaml
locales:
  default: en
  supported:
    - code: en
      label: English
      dir: ltr
      url_prefix: /
      date_format: "MMMM d, yyyy"

    - code: he
      label: עברית
      dir: rtl
      url_prefix: /he
      date_format: "d בMMMM yyyy"

    - code: ar
      label: العربية
      dir: rtl
      url_prefix: /ar
      date_format: "d MMMM yyyy"
```

The compiler should pass resolved locale data into templates.

Templates should not guess directionality.

## RTL support

RTL support affects both the public site and the admin editor.

The official theme should use logical CSS properties where possible:

```css
padding-inline-start
padding-inline-end
margin-inline-start
margin-inline-end
border-inline-start
text-align: start
```

Avoid hard-coding `left` and `right` unless required.

RTL support should be tested for:

- navigation
- breadcrumbs
- post cards
- pagination
- code blocks
- tables
- search UI
- editor UI
- media captions
- forms
- lifecycle banners
- date display

## Localization and AI

AI features should be locale-aware.

They should support:

- same-language rewriting
- translation
- SEO metadata in target locale
- localized excerpts
- tone adaptation per locale
- detecting untranslated content
- detecting translation drift
- RTL-aware preview review

The AI should receive explicit locale context.

Example:

```json
{
  "locale": {
    "code": "he",
    "dir": "rtl",
    "label": "עברית"
  },
  "task": "suggest_seo_description"
}
```

## Security model

The platform should not build its own password system.

The preferred model is:

> **Authentication is handled by Cloudflare Access or an equivalent identity layer. Authorization is handled by the CMS.**

This means:

- no CMS password database
- no CMS password reset flow
- no custom credential storage
- no local password hashing implementation
- no password-first login

The product should encourage:

- magic links or one-time codes
- identity provider login
- Cloudflare Access policies
- email/domain allowlists
- service tokens for automation

## Authentication vs authorization

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
What are you allowed to do in this CMS?
```

Cloudflare Access handles authentication.

The CMS handles authorization through D1.

Example request flow:

```text
User opens admin app
  ↓
Cloudflare Access authenticates user
  ↓
Worker receives validated identity
  ↓
CMS maps identity to user/site membership in D1
  ↓
CMS authorizes action
```

## Application roles

The v1 role model can be simple.

Suggested roles:

### Owner

Can:

- manage site settings
- manage users
- configure domains
- configure automations
- approve publishing
- connect Git or Cloudflare
- export project
- manage billing

### Editor

Can:

- create posts
- edit posts
- request publish
- approve AI suggestions if allowed
- manage media
- preview drafts

### Viewer

Can:

- view admin content
- view deployment history
- view workflow results

Enterprise roles can come later.

## Admin security

Admin APIs should require:

- validated Access identity
- site membership
- role-based authorization
- CSRF/session protections where applicable
- audit logging for sensitive actions
- rate limits for AI and automation endpoints

Sensitive actions include:

- publishing
- unpublishing
- changing lifecycle dates
- configuring automations
- connecting Git
- connecting domains
- exporting site
- changing roles
- activating deployments
- rolling back deployments

## Secrets

Secrets should never be stored in Git.

Examples:

- Git provider tokens
- Cloudflare API tokens
- AI provider keys
- webhook secrets
- source API credentials

Secrets should live in secure platform storage or Cloudflare bindings, depending on deployment mode.

Plugins and LLMs should not receive secrets directly.

## Public site security

The public site should be static whenever possible.

This reduces attack surface.

The public path should avoid:

- database reads
- session logic
- admin cookies
- dynamic rendering
- plugin runtime mutation

If runtime features are added later, such as comments or forms, they should be handled through explicit Worker endpoints or integrations.

## Cost model

One of the product promises is cheap serverless publishing.

Costs should be modeled by:

```text
views
actions
builds
storage
AI usage
admin seats
```

The architecture should make public traffic cheap and predictable.

## Cost dimensions

### Views

A view is public traffic.

A public page view may involve:

- HTML request
- asset requests
- CDN cache hits
- occasional R2 reads on cache miss
- optional lightweight Worker routing

Important rule:

> **Public views should not query D1.**

### Actions

An action is admin or workflow activity.

Examples:

- open editor
- save post
- upload media
- request AI suggestion
- generate draft
- run freshness automation
- build preview
- publish
- rollback

Actions may involve:

- Worker requests
- D1 reads/writes
- R2 reads/writes
- Git provider API calls
- AI usage
- build compute

### Builds

A build involves:

- reading content from Git
- evaluating lifecycle
- rendering templates
- running internal hooks
- generating RSS/sitemap/search index
- uploading output to R2
- recording deployment metadata

Build cost depends on site size and build runner architecture.

### Storage

Storage includes:

- media
- optimized media variants
- preview builds
- production deployments
- source snapshots
- logs
- generated reports

### AI usage

AI usage is separate because it depends on:

- provider
- model
- input tokens
- output tokens
- embeddings
- automation frequency

The CMS should show usage and enforce budgets.

## Rough cost formula

A simplified monthly cost model:

```text
monthly cost =
  workers
+ D1
+ R2 storage
+ R2 operations
+ build compute
+ AI usage
+ Access/admin seats, if applicable
```

More explicitly:

```text
views cost:
  public requests + cache misses + optional Worker routing

actions cost:
  admin API + D1 + Git + R2 + AI

builds cost:
  build compute + R2 writes + deployment metadata

storage cost:
  media + deployments + logs + snapshots

AI cost:
  tokens + embeddings + automation runs
```

## Cost-control principles

The platform should follow these rules:

1. **No public D1 reads.**  
   D1 is the control plane, not the public rendering layer.

2. **Cache public output aggressively.**  
   Static files should be served from the edge whenever possible.

3. **Batch lifecycle builds.**  
   If many posts transition at the same time, one build should handle them.

4. **Budget AI usage.**  
   AI actions and automations should have limits and visibility.

5. **Use lifecycle to reduce stale output.**  
   Unpublished content should leave search, RSS, sitemap, and public AI indexes.

6. **Keep generated content bounded.**  
   Generated posts should have lifecycle metadata by default.

7. **Make managed and self-hosted costs visible.**  
   Advanced users should understand what the platform is doing on their behalf.

## Pricing units for the product

The eventual SaaS pricing model can map to internal cost drivers:

- sites
- admin seats
- monthly views
- builds per month
- automations per month
- AI credits
- storage
- custom domains
- export/self-hosting features
- team permissions

The platform should track these from the beginning, even before final pricing is decided.

## What v1 should include

### Localization

- multiple locales
- RTL/LTR support
- locale-aware URLs
- locale-aware dates
- locale-specific lifecycle metadata
- one official RTL-compatible theme

### Security

- passwordless-first authentication
- Cloudflare Access or equivalent identity layer
- D1-based authorization
- no CMS-owned passwords
- audit logs for sensitive actions

### Costs

- usage tracking by site
- build count
- automation count
- AI usage logs
- storage estimate
- public traffic estimate if available
- admin action logs

## Principle

Security, localization, and cost control are not add-ons.

They are part of the core product promise:

> **A modern blogging platform should be globally usable, safe by default, and cheap enough to run without operational anxiety.**
