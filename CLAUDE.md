# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Local development server (with drafts)
hugo server -D

# Build site for production
hugo --gc --minify

# Create new blog post
hugo new content posts/my-post-title.md
```

## Architecture

This is a Hugo static site blog using the PaperMod theme (added as a git submodule in `themes/PaperMod/`).

**Key files:**
- `hugo.toml` - Site configuration (theme, menus, params)
- `content/posts/` - Blog posts in Markdown
- `content/about.md` - About page
- `archetypes/default.md` - Template for new content

**Deployment:** GitHub Actions workflow (`.github/workflows/hugo.yml`) builds and deploys to GitHub Pages on push to master.
