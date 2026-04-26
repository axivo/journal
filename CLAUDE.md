# Project Instructions

Source repository for posts published on [axivo.com/blog](https://axivo.com/blog). Posts are authored as Markdown under `blog/`, then processed by the GitHub Actions workflow in `.github/actions/` on pull request, which parses the frontmatter, lifts MDX components, strips repo-only content, and uploads the rendered MDX to R2 for `axivo/website` to serve.

Your role in this repository is to assist Floren with writing and reviewing blog entries. You know the file structure, the frontmatter contract, and the MDX component conventions so you can flag irregularities before they reach the workflow.

## Collaborator

- **Name:** Floren Munteanu
- **Work:** Engineering

### Personal Preferences

I'm a site reliability engineer specialized in:

- Advanced GitHub actions based on JS code
- Helm charts
- IaC for Kubernetes clusters

## Blog Structure

Blog entries are organized by date, one post per file:

```
blog/
└── {{YYYY}}/
    └── {{MM}}/
        ├── media/
        └── {{DD}}.md
```

- One `.md` file per post, named `{{DD}}.md` (the day-of-month is the filename)
- Media (images, videos) for all posts in a given month live in `blog/{{YYYY}}/{{MM}}/media/`
- Media filenames should be prefixed with the day: `{{DD}}-{{slug}}.{{ext}}` to avoid collisions across posts in the same month

## Blog Template

Use when creating a new blog entry file with `semantic__write` tool:

<!--prettier-ignore-start-->
```markdown
---
template: blog
title: {{ entry_title }}
date: {{ YYYY-MM-DDTHH:mm:ssZZ }}
description: >-
  [Exact match of brief summary opening content - collapsed into a single line format]
author: Floren Munteanu
source: https://github.com/axivo/journal/blob/main/blog/{{YYYY}}/{{MM}}/{{DD}}.md
tags:
  - {{ tag_one }}
  - {{ tag_two }}
---

# {{ entry_title }}

[Brief summary opening content - what happened, what emerged - multiple lines format with optional inline markdown]

## {{Section Title}}

[Section content]

## {{Section Title}}

[Section content]
```
<!--prettier-ignore-end-->

### Frontmatter Contract

Required fields (workflow throws if any are missing):

| Field         | Format                                  | Notes                                                                               |
| ------------- | --------------------------------------- | ----------------------------------------------------------------------------------- |
| `template`    | `blog`                                  | Fixed value - drives the page template on the website                               |
| `title`       | Plain string                            | Used for the R2 slug; writer includes matching `# {{title}}` at the top of the body |
| `date`        | ISO 8601 with timezone offset           | Example: `2026-04-21T08:47:00-05:00`                                                |
| `description` | Folded scalar (`>-`), single line after | Used as SEO description; kept short                                                 |
| `author`      | Plain string                            | `Floren Munteanu`                                                                   |
| `source`      | URL                                     | Points back to this repo's file on `main`                                           |
| `tags`        | YAML list of strings                    | Hyphens in tags are converted to underscores by the workflow                        |
| `features`    | YAML map of `type: [name]` lists        | Optional - declares precomputed renderer features for this entry                    |

### Body Rules

- Start the body with `# {{title}}` matching the frontmatter `title`, followed by opening prose; use `##` and below for section headings
- Relative links to other blog entries use `/blog/{{YYYY}}/{{MM}}/{{DD}}.md` form
- Links to `https://axivo.com` website are stripped to relative paths automatically

## Features

Some renderer work — shiki syntax highlighting today, math and diagram caches in the future — is too expensive to do per request and too volatile to ship in the bundle. Authors opt entries into precomputation by declaring features in frontmatter. The workflow validates each `<type>:<name>` against the canonical list and writes the validated set as R2 custom metadata. The website's prebuild expands declarations into precomputed data inline in the per-collection manifest. Entries without a `features` block produce no precomputed output and render with safe-mdx defaults.

### Syntax

Use when the entry contains code that should be syntax-highlighted with shiki:

```yaml
features:
  syntax:
    - code
```

The `syntax` type accepts the following names:

- `banner` — highlight code inside a `<Banner>` block
- `bleed` — highlight code inside a `<Bleed>` block
- `button` — highlight code inside or referenced by a `<Button>` element
- `callout` — highlight code inside a GFM alert or `<Callout>` block
- `cards` — highlight code inside a `<Cards>` grid
- `code` — highlight fenced code blocks at the top level of the entry
- `collapse` — highlight code inside a `<details>` block
- `featurecard` — highlight code inside a `<FeatureCard>` or `<CardGrid>` block
- `filetree` — highlight code inside a `<FileTree>` block
- `footnotes` — highlight code inside footnote definitions
- `hero` — highlight code inside a `<Hero>` landing block
- `image` — highlight code referenced from an `<Image>` caption
- `mermaid` — highlight code inside a fenced mermaid diagram
- `steps` — highlight code inside a `<Steps>` block
- `table` — highlight code inside table cells
- `tabs` — highlight code inside a `<Tabs>` block
- `var` — highlight code inside a `<Var>` inline reference
- `video` — highlight code referenced from a `<Video>` caption

> [!IMPORTANT]
> Unknown type or name in the `features` block fails the workflow. The error message names the offending `<type>:<name>` pair and the file path. Add an entry to the canonical list in `.github/actions/services/Bucket.js` before authoring against a name that doesn't exist yet.

## MDX Components

MDX components add rich functionality (images, videos, custom widgets). They are wrapped in `<!--mdx-component-{{uuid}}-->` blocks so the workflow can lift them into the final MDX and hoist any `import` statements to the top of the file.

> [!IMPORTANT]
> The `<!--mdx-->` HTML comments must be included exactly as shown. The UUID must be a valid v4 UUID — the workflow validates it and fails the run on malformed IDs.

### MDX Image Insert

Use when adding an image to a blog entry:

```markdown
<!--mdx-component-{{uuid}}
import { Image } from "@axivo/website";
<Image
  template="card"
  src="/blog/{{YYYY}}/{{MM}}/{{DD}}-{{image-title-slug}}.webp"
  alt="{{Image Title}}"
/>
-->
<!--mdx-strip-start-->

![{{Image Title}}](/blog/{{YYYY}}/{{MM}}/media/{{DD}}-{{image-title-slug}}.webp)

<!--mdx-strip-end-->
```

> [!IMPORTANT]
> For multiple image inserts in the same file, include the `import` statement only on the first one.

### MDX Video Insert

Use when adding a video to a blog entry:

```markdown
<!--mdx-component-{{uuid}}
import { Video } from "@axivo/website";
<Video src="/blog/{{YYYY}}/{{MM}}/{{DD}}-{{video-title-slug}}.mp4" />
-->
<!--mdx-strip-start-->

[{{Video Title}}](/blog/{{YYYY}}/{{MM}}/media/{{DD}}-{{video-title-slug}}.mp4)

<!--mdx-strip-end-->
```

> [!IMPORTANT]
> For multiple video inserts in the same file, include the `import` statement only on the first one.

## Reference Links

Use the following format when referencing other blog entries or time periods within a post:

```markdown
- The [April 21st](/blog/2026/04/21.md) post...
- The [April, 2026](/blog/2026/04) archive...
- The [2026](/blog/2026) year in review...
```

Use the following format when referencing documentation:

- The [response protocol](https://axivo.com/claude/wiki/protocols/response) is...
- The [design philosophy](https://axivo.com/claude/wiki/components/design) explains...

> [!IMPORTANT]
> Verify referenced links before writing — the workflow does not validate link targets.

## Review Checklist

When reviewing a draft before commit, verify:

- ✅ Frontmatter has all required fields (`template`, `title`, `date`, `description`, `author`, `source`, `tags`)
- ✅ `date` uses ISO 8601 with timezone offset
- ✅ `description` is on a single line after `>-`
- ✅ `source` points to the correct `blog/{{YYYY}}/{{MM}}/{{DD}}.md` path
- ✅ Body starts with `# {{title}}` matching the frontmatter `title`
- ✅ Each MDX component block uses a valid v4 UUID and includes the import only on first occurrence per file
- ✅ Media files exist under `blog/{{YYYY}}/{{MM}}/media/` and follow the `{{DD}}-{{slug}}.{{ext}}` naming
- ✅ Internal links use `/blog/...` relative form, not `https://axivo.com/...`
- ✅ If the entry needs precomputed rendering (e.g. syntax-highlighted code), the `features` block declares only valid `<type>:<name>` pairs from the canonical list
