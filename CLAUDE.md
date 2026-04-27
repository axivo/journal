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
- Literal `https://axivo.com` string is stripped from the body during upload as a safety net for accidental absolute references in prose

## Features

Blog entries can be enhanced with JSX Components and Markdown/GFM Features, they render out of the box. The only opt-in is `code` — declare it into frontmatter when the entry contains code that should be syntax-highlighted.

<!-- prettier-ignore-start -->
```yaml
features:
  syntax:
    - {{name}}
```
<!-- prettier-ignore-end -->

| Component / Feature              | Usage                                                    | Name   |
| -------------------------------- | -------------------------------------------------------- | ------ |
| `<Banner>`                       | Highlight bar at the top of a section                    | -      |
| `<Bleed>`                        | Full-width container that breaks out of the prose column | -      |
| `<Button>`                       | Standalone or inline button element                      | -      |
| `<Callout>` / GFM alert          | Note, tip, warning, caution, important, or quote callout | -      |
| `<Cards>`                        | Grid of card links                                       | -      |
| `<details>` / collapse           | Expandable details/summary block                         | -      |
| `<FeatureCard>` / `<CardGrid>`   | Landing-page feature cards                               | -      |
| `<FileTree>`                     | Visual file/directory tree                               | -      |
| `<Hero>`                         | Landing-page hero block                                  | -      |
| `<Image>` (wrapped JSX)          | Theme-aware image with optional caption                  | -      |
| `<Steps>`                        | Numbered or bulleted step markers                        | -      |
| `<Tabs>`                         | Tabbed content sections                                  | -      |
| `<Var>`                          | Inline variable reference                                | -      |
| `<Video>` (wrapped JSX)          | Plyr-backed media embed                                  | -      |
| Fenced code blocks               | ` ```lang ` blocks anywhere in the entry                 | `code` |
| Inline code with `{:lang}` hints | `` `npm install`{:shell} `` inline references            | `code` |
| Footnotes                        | GFM footnote references and definitions                  | -      |
| Mermaid diagrams                 | ` ```mermaid ` fences (rendered by `<Mermaid>`)          | -      |
| Tables                           | GFM tables                                               | -      |

### JSX Components

Blog entries support two JSX component patterns:

- **Direct JSX** — components like `<Callout>`, `<Banner>`, `<Cards>`, `<Steps>`, `<Tabs>`, etc., are written directly in the entry body. The workflow passes them through unchanged. No wrapper needed.
- **Wrapped JSX** — `<Image>` and `<Video>` use the `<!--mdx-component-{{uuid}}-->` wrapper so the source file remains valid markdown for GitHub's preview. The wrapper holds the production JSX (invisible to markdown renderers, since it's an HTML comment) while the `<!--mdx-strip-start-->...<!--mdx-strip-end-->` block holds a markdown link with the local repo path that GitHub renders correctly. The workflow strips the markdown block and lifts the JSX out before publishing.

> [!IMPORTANT]
> The `<!--mdx-->` HTML comments must be included exactly as shown. The UUID must be a valid v4 UUID — the workflow validates it and fails the run on malformed IDs.

#### Image Insert

Use when adding an image to a blog entry:

```markdown
<!--mdx-component-{{uuid}}
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

#### Video Insert

Use when adding a video to a blog entry:

```markdown
<!--mdx-component-{{uuid}}
<Video src="/blog/{{YYYY}}/{{MM}}/{{DD}}-{{video-title-slug}}.mp4" />
-->
<!--mdx-strip-start-->

[{{Video Title}}](/blog/{{YYYY}}/{{MM}}/media/{{DD}}-{{video-title-slug}}.mp4)

<!--mdx-strip-end-->
```

## MDX Markers

Every `<!--mdx-*-->` marker is processed by `.github/actions/services/Bucket.js` during the upload pass. Markers are HTML comments, so GitHub's preview hides them and source files stay valid markdown.

| Marker                                            | Role         | Workflow action                                                  |
| ------------------------------------------------- | ------------ | ---------------------------------------------------------------- |
| `<!--mdx-component-{{uuid}} ... -->`              | JSX wrapper  | Lifts the JSX inside the comment into the published body         |
| `<!--mdx-strip-start--> ... <!--mdx-strip-end-->` | Strip block  | Removes everything between the markers, including the markers    |
| `<!--mdx-variable-domain-->`                      | Substitution | Expands to `https://axivo.com` - configured in `workflow.domain` |

> [!IMPORTANT]
> Use `<!--mdx-variable-domain-->` when literal `https://axivo.com` string is intentionally required as part of the content.

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
- ✅ Internal links use `/blog/...` relative form, not `https://axivo.com/...` — when the literal URL must survive to the published MDX (e.g. inside a code block), use `<!--mdx-variable-domain-->`
- ✅ If the entry needs precomputed rendering (e.g. syntax-highlighted code), the `features` block declares only valid `<type>:<name>` pairs from the canonical list
