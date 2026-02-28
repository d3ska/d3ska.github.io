# Matthew on Software — Project Context

## Tech Stack
- **Framework:** Jekyll with jekyll-theme-chirpy (~7.0)
- **Markup:** Markdown (kramdown engine), Liquid templates
- **Syntax highlighting:** Rouge (line numbers for code blocks)
- **Comments:** Utterances (GitHub issues-based)
- **Deployment:** GitHub Actions → GitHub Pages
- **Domain:** mattonsoftware.com

## Project Structure
```
_posts/           → Published blog posts
in_progress/      → Work-in-progress drafts
_invisible/       → Hidden/unpublished posts
removed/          → Archived posts
_layouts/         → HTML templates
_includes/        → Reusable HTML fragments
_tabs/            → Static pages (about, archives, categories, tags)
assets/images/    → Profile images, avatars
assets/blog_images/[DATE-TITLE]/  → Post-specific images
```

## Post Format
- **Filename:** `YYYY-MM-DD-post-title.md`
- **Front matter:**
  ```yaml
  ---
  title: "Post Title"
  categories:
    - Category Name
  tags:
    - Tag 1
    - Tag 2
  ---
  ```
- Layout, comments, toc, and permalink are inherited from `_config.yml` defaults

## Blog Categories
Design Patterns, Testing, Java/Spring, Software Design Principles, Data Structures & Algorithms, System Design, Concurrency

## Build Commands
- `bundle install` — install dependencies
- `bundle exec jekyll s` — local dev server
- `bundle exec jekyll b -d "_site"` — production build

## Git Workflow
- **Never push to remote** — only commit locally
- **Group related changes** into logical commits with descriptive messages
- **Do not add** `Co-Authored-By: Claude` to commit messages

## Common Pitfalls
- Watch for missing English articles (a, an, the) — recurring issue across posts
- Prefer code blocks over code in images
- Ensure code examples compile and are correct

## Writing Style

### Voice & Tone
- First person ("I", "my") used naturally, not forced
- Conversational but professional. Don't dumb things down but don't show off either
- Honest about trade-offs and limitations. Say when something is debated or imperfect
- Personal anecdotes appear occasionally ("I have personally refactored...", "Trust me")

### Structure
- Problem-first approach: start with the problematic scenario, then introduce the solution
- `###` for main sections, `####` for subsections
- Bullet points for lists of characteristics, pros/cons
- Bold on first use of key terms
- Posts follow a consistent flow: why it matters, what it is, code example, when to use / when not to use

### Code Examples
- Complete, practical Java or Go examples (not toy snippets)
- Often show "bad" then "good" implementation side by side
- Use `// Given`, `// When`, `// Then` comments in tests
- Minimal comments inside code otherwise, let the code speak

### Paragraphs & Sentences
- Mix of short and longer sentences. Not rigidly short
- Paragraphs are typically 2-5 sentences, whatever the point needs
- Use whitespace generously between sections

### Things to Avoid
- No em dashes (use commas, periods, colons, or parentheses instead)
- No excessive hedging or filler words
- No "Let's dive in" / "In this article we will" type intros
- No emojis

### Recurring Patterns
- Real-world analogies to explain abstract concepts
- Cross-links to related posts via `> **Related posts**:` blockquotes
- Acknowledge when something is debated or controversial rather than presenting opinions as facts
- "When to use" / "When not to use" practical guidance sections
- Light/dark image pairs for diagrams
