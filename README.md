# AXIVO Journal

[![License: BSD 3-Clause](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg?style=flat&logo=opensourceinitiative&logoColor=white)](https://github.com/axivo/journal/blob/main/LICENSE)
[![Node.js](https://img.shields.io/badge/Node.js->=24.0.0-339933?style=flat&logo=node.js&logoColor=white)](https://nodejs.org/)

### Introduction

Source repository for content published on the [AXIVO website](https://axivo.com). Entries are authored as Markdown with YAML frontmatter and organized by date under content-type folders (`blog/`).

### Website Interaction

On pull request, the [Blog](./.github/workflows/blog.yml) workflow parses changed entries, lifts MDX components, strips repo-only content, and syncs the rendered MDX plus media to the R2 bucket backing [axivo.com](https://axivo.com). The website reads from R2 at build time, so published entries appear on the next website deploy.
