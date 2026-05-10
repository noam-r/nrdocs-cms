# AI & Automations

## AI as a scoped processor

LLMs can make the CMS significantly more useful, but they should follow the same principles as the rest of the system.

The model is:

```text
structured context in
  ↓
LLM processing
  ↓
structured output out
  ↓
core validation
  ↓
user or policy approval
  ↓
Git change
```

The LLM should not own state, access secrets, mutate files directly, publish sites directly, or become the source of truth.

## Core doctrine

nrdocs CMS should treat AI as a controlled assistant.

```text
AI receives context, not access.
AI returns suggestions, not mutations.
AI output is structured, not free-form.
AI changes are reviewed, validated, and committed.
AI behavior is auditable.
Git remains the source of truth.
```

This makes AI useful without making the system opaque.

## AI output types

AI features should return a small set of structured outputs.

### Diagnostics

Warnings, suggestions, or review findings.

Example:

```json
{
  "diagnostics": [
    {
      "level": "warning",
      "message": "This post references an upcoming launch, but the lifecycle fresh_until date has passed.",
      "path": "/body/sections/2"
    }
  ]
}
```

### Patches

Suggested changes to content or metadata.

Example:

```json
{
  "patches": [
    {
      "op": "add",
      "path": "/frontmatter/description",
      "value": "Learn why static blogs often go stale and how lifecycle-aware publishing can help."
    }
  ]
}
```

### Artifacts

Generated files or reports.

Example:

```json
{
  "artifacts": [
    {
      "path": ".nrdocs/reports/freshness-audit.json",
      "contentType": "application/json",
      "content": "{...}"
    }
  ]
}
```

The core validates these outputs before applying them.

## Editor AI

The first AI use cases should be practical writing assistance.

Examples:

- rewrite selected text
- shorten a paragraph
- make a section clearer
- make a post more technical
- make a post more beginner-friendly
- generate title options
- generate excerpt
- generate SEO description
- suggest tags
- turn rough notes into a draft
- review tone and clarity

These actions should operate on scoped context, such as:

- selected text
- current post
- current locale
- style guide
- existing frontmatter

The user should approve changes before they are committed.

## Metadata AI

Blog posts need metadata.

AI can suggest:

- title alternatives
- slug suggestions
- excerpt
- SEO description
- OpenGraph title
- OpenGraph description
- tags
- category
- related posts
- translation summary

Example flow:

```text
editor requests metadata suggestions
  ↓
CMS sends post title, body excerpt, locale, and existing metadata
  ↓
LLM returns structured metadata suggestions
  ↓
editor accepts or rejects each suggestion
  ↓
accepted changes become a Git patch
```

## Freshness review

Lifecycle makes AI more valuable.

When a post reaches `fresh_until` or `review_at`, the CMS can ask an LLM to review it.

Possible review questions:

- Does this post contain time-sensitive claims?
- Does it refer to future events that may have passed?
- Does it need updated examples?
- Does it reference old product names?
- Does the tone still match the site style guide?
- Are there missing caveats?
- Should the post be updated, archived, or left as-is?

The output should be diagnostics and optional patches.

## Localization AI

Localization should be supported from day one.

AI can assist with:

- translating posts
- preserving tone across locales
- rewriting for a specific locale
- detecting untranslated content
- detecting translation drift
- localizing SEO metadata
- checking RTL-specific layout previews

Each locale should have its own content file and lifecycle metadata.

AI should know the target locale and direction.

Example:

```json
{
  "locale": {
    "code": "he",
    "dir": "rtl"
  },
  "task": "suggest_seo_metadata"
}
```

## Automations

Automations are scheduled or event-driven workflows.

They can:

- publish scheduled posts
- unpublish expired posts
- run stale-content reviews
- generate drafts from sources
- create weekly roundup posts
- summarize deployment changes
- notify editors
- trigger preview builds
- trigger production builds

Automations should produce Git changes before affecting public content.

## Freshness automations

A freshness automation keeps content up to date.

Example:

```text
schedule
  ↓
fetch trusted sources
  ↓
normalize records
  ↓
LLM summarizes or transforms
  ↓
validate output
  ↓
create draft post or patch
  ↓
build preview
  ↓
editor approves
  ↓
merge to production branch
  ↓
deploy static output
```

The public site stays static.

The content changes over time through scheduled, auditable workflows.

## Source-grounded generation

For factual content, the LLM should not be the source of truth.

The source data is the source of truth.

The LLM transforms supplied source material.

Bad:

```text
Ask the model what happened today.
```

Good:

```text
Fetch approved RSS feeds.
Fetch source articles or excerpts.
Ask the model to summarize only the supplied sources.
Require source links in the output.
```

This is especially important for news, briefings, financial content, legal content, health content, and political content.

## First freshness automation

The v1 product should include one simple, safe freshness automation.

Recommended first automation:

> **RSS/source links to draft post**

Flow:

```text
user configures sources
  ↓
automation runs daily or weekly
  ↓
CMS fetches source items
  ↓
LLM summarizes and groups them
  ↓
CMS creates a draft post
  ↓
editor reviews and approves
  ↓
post is committed to Git and deployed
```

This proves the “up-to-date CMS” concept without launching directly into fully autonomous news publishing.

## Automation output targets

Automations should update clearly defined targets.

Possible targets:

### Draft post

Creates a new post that requires approval.

Best for:

- weekly roundups
- industry briefs
- newsletter drafts
- event recaps

### Generated post

Creates a post in a generated content folder.

Best for:

- daily briefings
- source-grounded summaries
- recurring reports

### Generated block

Updates a specific block inside a page or post.

Best for:

- weather snippets
- status summaries
- temporary notices
- regularly refreshed metadata

### Review task

Creates diagnostics and asks a human to act.

Best for:

- stale content checks
- legal/policy review
- high-risk topics

## Generated content ownership

Generated blocks should be explicit.

Example:

```markdown
<!-- nrdocs:generated id="weekly-summary" owner="job:weekly-roundup" -->
Generated content appears here.
<!-- /nrdocs:generated -->
```

Rules:

- only the owning job can update the block automatically
- humans should be warned before editing inside generated regions
- automations should not modify arbitrary authored content
- conflicting changes should create a review task

## Approval policies

Not every automation should behave the same way.

Suggested approval modes:

```text
draft_only
approval_required
auto_if_validation_passes
auto_publish
```

Recommended v1 defaults:

- AI writing suggestions require approval
- generated news or roundup posts require approval
- lifecycle publish/unpublish can run automatically
- low-risk metadata suggestions require approval before applying
- fully autonomous publishing should be opt-in

## Validation

Every AI or automation output should be validated.

Validation can include:

- schema validation
- required fields
- source citation checks
- maximum length
- banned terms
- locale checks
- lifecycle checks
- link validation
- duplicate detection
- build success
- safety checks for high-risk topics

If validation fails, the system should not publish.

It should create diagnostics or a review task.

## Provenance

Generated or AI-assisted content should include provenance.

Possible metadata:

```yaml
ai:
  assisted: true
  last_reviewed_at: 2026-05-08T09:20:00+03:00

generated:
  by: weekly-roundup
  run_id: run_123
  generated_at: 2026-05-08T09:00:00+03:00
  sources:
    - https://example.com/source
```

The full workflow log can live in D1 and R2, but the content file should contain enough metadata to explain its origin.

## Audit trail

The CMS should record:

- who requested an AI action
- which workflow ran
- what context was sent
- which model/provider was used
- what output was returned
- which patches were accepted
- who approved them
- which commit included them
- which deployment went live

This matters for trust.

## AI and cost control

AI usage should be explicit and budgeted.

The CMS should track:

- tokens
- model/provider
- action type
- user
- site
- automation
- estimated cost
- monthly limits

Automations should have budgets and rate limits.

A user should understand when a workflow may cost money.

## Public AI features

Public-facing AI, such as “Ask this blog,” should not be part of v1.

It introduces:

- runtime cost
- abuse risk
- data exposure questions
- retrieval index management
- answer quality concerns
- latency

It can be added later as an optional runtime feature.

The first version should focus on build-time and editor-time AI.

## What v1 should include

The first version should include:

- title suggestions
- excerpt generation
- SEO description generation
- tag suggestions
- selected text rewrite
- post clarity review
- stale content review
- RSS/source-to-draft automation
- deployment summary
- AI usage logging

## What v1 should avoid

The first version should avoid:

- autonomous open-web news publishing
- public AI chatbot
- unreviewed AI commits
- AI with access to admin/user data
- AI with access to secrets
- large-scale autogenerated SEO pages
- runtime AI personalization

## Principle

AI should make the CMS smarter without making it less trustworthy.

> **The system should use AI to assist content operations, not to replace the source of truth.**
