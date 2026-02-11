---
name: readme-editor
description: "Use this skill when editing, generating, or improving a README.md file. Triggers: any mention of 'README', 'readme', documentation for a project, project description, badges, table of contents, or when the user asks to document a codebase. Also triggers when analyzing an existing README for improvements. Covers: structure, badges, TOC, feature sections, tech stack tables, installation guides, contributing guidelines, license sections, and bilingual READMEs."
---

# README.md Editor & Generator

## Philosophy

A README is the front door of your project. It should answer three questions in under 30 seconds:
1. **What** does this project do?
2. **Why** should I care?
3. **How** do I get started?

Everything else is progressive disclosure.

## Before You Start

1. **Read the existing README** (if any) with the Read tool — never overwrite blindly
2. **Scan the codebase** to extract real data:
   - `cat package.json` → name, version, description, scripts, dependencies
   - `ls src/` → project structure
   - `git remote -v` → repo URL for badges
   - `cat LICENSE` → license type
   - `cat .env.example` → required environment variables
3. **Identify the audience**: open-source contributors, team members, end users, recruiters?

## README Structure (Canonical Order)

```markdown
# Project Name

Short one-liner description (matches package.json description).

![Hero image or demo GIF](link)    ← optional, high impact

## Badges (optional)
## Features
## Demo / Live Site
## Tech Stack
## Getting Started
### Prerequisites
### Installation
### Environment Variables
### Running Locally
## Usage / Examples
## Project Structure
## Scripts Reference
## API Reference (if applicable)
## Roadmap
## Contributing
## License
## Acknowledgments
```

Not every project needs every section. Use judgment:
- **Portfolio / personal project** → Features, Tech Stack, Getting Started, License
- **Open-source library** → All sections, especially API Reference and Contributing
- **Internal tool** → Getting Started, Scripts, Environment Variables, Project Structure

## Section-by-Section Guidelines

### Title & Description
```markdown
# Project Name

One-sentence description. No marketing fluff. State what it does.
```
- Extract from `package.json` `description` field if available
- Never start with "This is a..." — just state what it does
- Max 2 lines

### Badges
Generate badges from real project data. Use shields.io format:
```markdown
![Build](https://img.shields.io/github/actions/workflow/status/USER/REPO/ci.yml?branch=main)
![License](https://img.shields.io/github/license/USER/REPO)
![Version](https://img.shields.io/github/package-json/v/USER/REPO)
![Node](https://img.shields.io/badge/node-%3E%3D18-brightgreen)
```

Common badge categories:
- **Build status**: CI/CD pipeline status
- **Version**: package.json version or latest release
- **License**: from LICENSE file
- **Tech**: Node version, framework version
- **Quality**: test coverage, code quality
- **Activity**: last commit, open issues

Rules:
- Only add badges for things that actually exist (don't add coverage badge if no tests)
- Max 6 badges — more than that is noise
- Order: build → version → license → tech → quality

### Features
```markdown
## Features

- **Feature name** — Short description of what it does and why it matters
- **Another feature** — Concrete benefit, not vague marketing
```

Rules:
- Scan the codebase to list REAL features, not aspirational ones
- Use bold for feature name, dash, then description
- Each feature = 1 line (not a paragraph)
- Group related features if > 8 items
- If the project has i18n, mention supported languages

### Tech Stack
Use a table for readability:
```markdown
## Tech Stack

| Category      | Technologies                          |
|---------------|---------------------------------------|
| Framework     | Astro 5, React 19                     |
| Styling       | Tailwind CSS 3.4, CSS custom props    |
| Content       | Markdown/MDX, Zod schema validation   |
| Deployment    | GitHub Pages, GitHub Actions           |
```

Rules:
- Extract real dependencies from `package.json`
- Include version numbers for major frameworks
- Group by category: Framework, Styling, State, Testing, Deployment, etc.
- Don't list every dev dependency — focus on architectural choices

### Getting Started

```markdown
## Getting Started

### Prerequisites

- Node.js >= 18
- pnpm (recommended) or npm

### Installation

git clone https://github.com/USER/REPO.git
cd REPO
pnpm install

### Environment Variables

Copy `.env.example` to `.env` and fill in the values:

cp .env.example .env

| Variable          | Description              | Required |
|-------------------|--------------------------|----------|
| `DATABASE_URL`    | PostgreSQL connection    | Yes      |
| `API_KEY`         | Third-party API key      | No       |

### Running Locally

pnpm dev
```

Rules:
- Extract scripts from `package.json` — use the ACTUAL commands
- If `.env.example` exists, document every variable
- Always specify the package manager the project uses
- Include the port number if it's a web project

### Project Structure
```markdown
## Project Structure

src/
├── components/    # Reusable UI components
├── layouts/       # Page layouts
├── pages/         # Route pages (file-based routing)
├── content/       # MDX content collections
├── styles/        # Global styles and Tailwind config
├── lib/           # Utility functions and helpers
└── assets/        # Static assets (images, fonts)
```

Rules:
- Generate from actual `ls` / `find` output — never invent structure
- Only show 2 levels deep max
- Add brief comment for each directory
- Skip `node_modules`, `.git`, `dist`, `build`

### Scripts Reference
```markdown
## Scripts

| Command           | Description                    |
|-------------------|--------------------------------|
| `pnpm dev`        | Start dev server on :4321      |
| `pnpm build`      | Production build to `./dist`   |
| `pnpm preview`    | Preview production build       |
| `pnpm lint`       | Run ESLint                     |
| `pnpm test`       | Run test suite                 |
```

Rules:
- Extract ALL scripts from `package.json`
- Add the port number for dev/preview
- Mention output directory for build

### License
```markdown
## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.
```
- Read the actual LICENSE file to determine the type
- Link to the file, don't paste the full text

## Editing an Existing README

When asked to edit (not generate from scratch):

1. **Read the full current README** with the Read tool
2. **Identify what's wrong or missing** — compare against the canonical structure
3. **Preserve the author's voice and style** — don't rewrite everything
4. **Use targeted edits** with the Edit tool — don't replace the whole file
5. **Keep custom sections** the author added — they're intentional

Common edit requests and how to handle them:
- "Update the README" → Scan codebase for changes, update outdated sections
- "Add badges" → Check CI, license, package.json for real badge data
- "Improve the README" → Check structure order, missing sections, outdated info
- "Make it bilingual" → See Bilingual README section below
- "Add table of contents" → Generate from actual headings

## Bilingual README

Two approaches depending on content length:

**Approach A — Language switcher (recommended for shorter READMEs):**
```markdown
# Project Name

🇬🇧 [English](#english) | 🇫🇷 [Français](#français)

---

## English

[Full content in English]

---

## Français

[Full content in French]
```

**Approach B — Separate files (for longer READMEs):**
```
README.md          ← Default language (usually English)
README.fr.md       ← French version
```
With a link at the top of each:
```markdown
🇫🇷 [Version française](README.fr.md)
```

Rules:
- NEVER use machine-translation-style French — write natural, technical French
- Keep technical terms in English when they're standard (e.g., "pull request", "commit", "deploy")
- Badge URLs stay identical in both versions
- Translate section headers but keep anchor links working

## Table of Contents Generation

Generate TOC from actual headings in the file:

```markdown
## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
- [Project Structure](#project-structure)
- [License](#license)
```

Rules:
- Only generate TOC if README has > 5 sections
- Use GitHub-compatible anchor links (lowercase, hyphens, no special chars)
- Indent for h3 subsections
- Don't include h1 (title) or the TOC itself in the TOC

## Quality Checklist

Before finishing any README edit, verify:

- [ ] All links work (no placeholder URLs)
- [ ] Code blocks have language identifiers (```bash, ```json, etc.)
- [ ] Commands are copy-pasteable (no `$` prefix, no `>` prefix)
- [ ] Version numbers match `package.json`
- [ ] Screenshots/GIFs actually exist at referenced paths
- [ ] No "Lorem ipsum" or "TODO" left in the file
- [ ] License matches the actual LICENSE file
- [ ] Repo URL is correct (check `git remote -v`)
- [ ] `.env.example` variables are all documented
- [ ] Scripts match actual `package.json` scripts

## Anti-Patterns

- ❌ Writing a README without scanning the codebase first
- ❌ Adding badges for things that don't exist (coverage badge with no tests)
- ❌ Pasting entire LICENSE text instead of linking
- ❌ Using `$` or `>` before shell commands (breaks copy-paste)
- ❌ Marketing language ("revolutionary", "blazing fast", "cutting-edge")
- ❌ Documenting features that aren't implemented yet without marking them
- ❌ Screenshots that don't match the current UI
- ❌ Hardcoding usernames/URLs that should be dynamic
