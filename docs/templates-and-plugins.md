# Templates & Plugins

## Extensibility without WordPress-style runtime risk

WordPress became popular partly because of its themes and plugins.

They let users create flexible websites without building everything from scratch.

nrdocs CMS should preserve that spirit of extensibility, but with a different security and architecture model.

The goal is:

> **Flexible presentation and extension points without arbitrary runtime mutation.**

Templates and plugins should follow the same core principle:

```text
structured input
  ↓
controlled processing
  ↓
structured output
  ↓
core validation
```

## Templates are passive renderers

Templates should not query databases, read arbitrary files, call APIs, or mutate state.

They receive structured site data from the compiler and render static output.

```text
site data
  ↓
template
  ↓
HTML/assets
```

The core owns:

- content loading
- lifecycle evaluation
- localization
- collection building
- routing
- search index generation
- plugin hook execution
- deployment

Templates own:

- layout
- visual structure
- reusable partials
- theme styling
- presentation of lifecycle states
- locale-aware rendering

## Template data model

A template should receive only the data it needs.

Example post template context:

```json
{
  "site": {
    "title": "Example Blog",
    "url": "https://example.nrdocs.site"
  },
  "locale": {
    "code": "en",
    "dir": "ltr"
  },
  "page": {
    "title": "Why Static Blogs Go Stale",
    "slug": "/blog/why-static-blogs-go-stale",
    "content": "<p>...</p>",
    "description": "...",
    "tags": ["cms", "static-sites"],
    "lifecycle": {
      "state": "active_fresh",
      "freshUntil": "2026-06-08T09:30:00+03:00"
    }
  }
}
```

The template should not decide whether the page is visible.

The compiler decides that.

The template may display a stale warning if the compiler says the page is stale.

## Suggested template types

A v1 blogging platform needs a focused set of templates:

- home/index
- post
- page
- tag archive
- category archive
- author archive
- date archive
- RSS
- sitemap
- 404

This keeps the first theme system focused while still supporting a complete blog.

## Template language

The first version should prefer a restricted template language.

Good candidates conceptually:

- Liquid-like
- Handlebars-like
- Nunjucks-like with restrictions

The template language should support:

- variables
- loops
- conditionals
- partials
- blocks
- filters
- escaping
- slots

It should avoid:

- arbitrary JavaScript execution
- filesystem access
- network access
- database queries
- environment variable access
- unbounded recursion
- secret access

The system can add more advanced rendering later, but v1 should prioritize safety and portability.

## Theme package shape

A theme can be represented as files in Git.

Example:

```text
theme/
  tokens.yml
  layouts/
    base.html
    post.html
    page.html
    index.html
  partials/
    header.html
    footer.html
    post-card.html
    pagination.html
  components/
    callout.html
    newsletter-signup.html
  styles/
    theme.css
```

A theme manifest can describe available layouts, components, slots, and tokens.

Example:

```yaml
name: minimal-blog
version: 1.0.0
api: nrdocs-theme/v1

layouts:
  - id: post
    file: layouts/post.html
  - id: page
    file: layouts/page.html
  - id: index
    file: layouts/index.html

partials:
  - id: header
    file: partials/header.html
  - id: footer
    file: partials/footer.html

components:
  - id: callout
    file: components/callout.html
    fields:
      type:
        type: enum
        options: [info, warning, danger, success]
      title:
        type: string
      body:
        type: markdown
```

## Design tokens

Users should be able to customize a theme without editing templates.

Design tokens can live in Git.

Example:

```yaml
theme:
  colors:
    primary: "#2563eb"
    background: "#ffffff"
    text: "#111827"

  typography:
    heading_font: Inter
    body_font: Inter
    code_font: JetBrains Mono

  layout:
    max_width: 1120px
    sidebar: false
```

The CMS can expose these as visual controls.

This gives users a simple customization layer while preserving a clean file-based theme.

## Components

Components are reusable content blocks.

Examples:

- callout
- hero
- feature grid
- image gallery
- newsletter signup
- related posts
- table of contents
- code group
- tabs
- accordion

Components can be used inside Markdown.

Example:

```markdown
::callout
type: warning
title: Time-sensitive content
body: This post should be reviewed after the election date.
::
```

The component schema allows the CMS editor to render friendly forms instead of forcing users to edit raw syntax.

## Slots

Slots are controlled extension areas inside templates.

Example:

```html
<header>
  {{ slot "header.before" }}
  {{ partial "nav" }}
  {{ slot "header.after" }}
</header>

<article>
  {{ page.content }}
  {{ slot "post.afterContent" }}
</article>
```

Slots can be filled by theme configuration, plugins, or site settings.

They are similar to WordPress widget areas, but they are explicit, static, and controlled.

## Template inheritance

The system should support inheritance or composition.

Example:

```html
{{ extends "base.html" }}

{{ block "main" }}
  <article>
    <h1>{{ page.title }}</h1>
    {{ page.content }}
  </article>
{{ end }}
```

This makes themes easier to maintain.

Child overrides can live in Git.

Example:

```text
theme/
  overrides/
    partials/footer.html
    layouts/post.html
```

## Static queries, not runtime queries

Templates may need lists of posts, tags, or related content.

They should not query a database at runtime.

Instead, the compiler provides static collections.

Example template data:

```json
{
  "collection": {
    "type": "posts",
    "items": [
      {
        "title": "Post title",
        "url": "/blog/post-title",
        "excerpt": "..."
      }
    ]
  }
}
```

The query runs during build.

The output is static.

## Plugins are passive hook subscribers

Plugins should not have direct access to the CMS internals.

They should subscribe to hook points and receive structured payloads.

```text
core emits hook payload
  ↓
plugin receives limited input
  ↓
plugin returns structured output
  ↓
core validates output
  ↓
core applies or rejects result
```

The plugin does not query D1.

The plugin does not read arbitrary repository files.

The plugin does not write to R2.

The plugin does not deploy.

The plugin does not mutate global state.

## Hook-based permissions

Permissions are defined by hook subscriptions.

A plugin declares which hooks it wants.

Example:

```yaml
name: nrdocs-plugin-seo
version: 1.0.0
api: nrdocs-hooks/v1

subscriptions:
  - hook: content.frontmatter.validate@v1
  - hook: render.page.after@v1
```

Each hook defines:

- when it runs
- what data it receives
- what output it may return
- whether it can block the workflow
- whether third-party plugins may subscribe
- the sensitivity level of its payload

This is simpler and safer than broad permissions such as “read all content” or “write files.”

## Hook result types

Plugins should return the same kinds of structured outputs as AI features.

### Diagnostics

```json
{
  "diagnostics": [
    {
      "level": "warning",
      "message": "Missing SEO description.",
      "path": "/frontmatter/description"
    }
  ]
}
```

### Patches

```json
{
  "patches": [
    {
      "op": "add",
      "path": "/frontmatter/description",
      "value": "A concise summary of the post."
    }
  ]
}
```

### Artifacts

```json
{
  "artifacts": [
    {
      "path": "feed.xml",
      "contentType": "application/xml",
      "content": "..."
    }
  ]
}
```

The core validates every result before applying it.

## Example hook categories

### Content hooks

- content.discovered
- content.frontmatter.validate
- content.markdown.transform
- content.references.resolve
- content.collection.validate

### Render hooks

- render.page.before
- render.page.after
- render.html.transform
- render.search.index

### Build hooks

- build.started
- build.graph.created
- build.beforeEmit
- build.completed
- build.failed

### Lifecycle hooks

- lifecycle.evaluated
- lifecycle.stale
- lifecycle.review_due
- lifecycle.unpublished

### Deployment hooks

- deploy.preview.completed
- deploy.production.completed
- deploy.failed

## Risk levels

Hooks should have sensitivity levels.

Example:

| Level | Meaning |
| --- | --- |
| 0 | No content, lifecycle event only |
| 1 | Public metadata |
| 2 | Single document content |
| 3 | Full public content graph |
| 4 | Admin/control-plane metadata |
| 5 | Secrets, billing, auth; unavailable to plugins |

A plugin install screen can explain exactly what the plugin receives.

## Plugin execution

The hook protocol can support several execution modes over time:

- internal first-party plugins
- local build plugins
- sandboxed JavaScript
- WASM plugins
- remote HTTP hook endpoints
- marketplace plugins later

For v1, public third-party plugins can be postponed.

The internal architecture should use hooks, but the ecosystem does not need to launch on day one.

## Recommended v1 plugin stance

v1 should include internal hook-based features:

- SEO metadata
- RSS generation
- sitemap generation
- lifecycle evaluator
- AI reviewer
- static search index
- link checker, if simple

But v1 should not yet include:

- public plugin marketplace
- arbitrary runtime plugins
- plugin-owned D1
- plugins that modify authentication
- plugins that modify deployment core
- plugins with unrestricted network access

## Difference from WordPress plugins

WordPress plugins are powerful partly because they can execute inside the application runtime.

That power also creates risk.

nrdocs plugins should be powerful because they operate at clear extension points.

| WordPress plugin model | nrdocs plugin model |
| --- | --- |
| Arbitrary runtime code | Hook-based structured processing |
| Shared application authority | No ambient authority |
| Direct database access | No direct D1 access |
| Can mutate global state | Core validates outputs |
| Runtime performance risk | Mostly build/workflow-time execution |
| Hard to audit | Hook payloads and outputs are auditable |

## Principle

Templates and plugins should make the system flexible without making it unpredictable.

> **Templates render structured data. Plugins transform structured hook payloads. The core remains in control.**
