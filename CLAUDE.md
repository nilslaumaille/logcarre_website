# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LogCarré is a Hugo static site built on the [Hugoplate](https://github.com/zeon-studio/hugoplate) boilerplate. It uses Hugo Modules, Tailwind CSS v4, and supports French (primary) and English (secondary) content languages.

## Commands

```bash
npm run dev          # Start Hugo development server
npm run build        # Production build (minified, GC, metrics)
npm run preview      # Preview production build with live reload
npm run format       # Run Prettier on all files
npm run update-modules  # Update all Hugo modules to latest
npm run update-theme    # Update the hugoplate theme
```

**Requirements**: Hugo Extended v0.144+, Node.js v22+, Go v1.24+

## Architecture

### Project Setup Script
Running `npm run project-setup` reorganizes the repo: it moves `layouts/`, `assets/`, and `tailwind-plugin/` into `themes/hugoplate/`, and copies `exampleSite/` content to the root. The current repo has already been through this step — files you see deleted in git status were moved into `themes/hugoplate/`.

### Multilingual Configuration
- French (`fr`) is the primary language, English (`en`) is secondary
- `defaultContentLanguageInSubdir = true` means all URLs include language prefix (e.g., `/fr/`, `/en/`)
- Language-specific config files: `config/_default/params.en.toml`, `config/_default/params.fr.toml`, `config/_default/menus.en.toml`, `config/_default/menus.fr.toml`
- Content lives in `content/french/` and `content/english/`
- i18n strings: `i18n/en.yaml`, `i18n/fr.yaml`

### Design Tokens System
`data/theme.json` is the single source of truth for colors and fonts. The `tailwind-plugin/tw-theme.js` plugin reads this file at build time and generates CSS custom properties. To change site colors or typography, edit `theme.json` — do not add raw CSS color values.

### CSS Architecture
- `assets/css/custom.css` — custom styles, use Tailwind `@layer` directives here
- Theme layouts are in `themes/hugoplate/assets/css/`
- Tailwind processes CSS through Hugo Pipes; no separate PostCSS step needed
- `tailwind-plugin/tw-bs-grid.js` provides Bootstrap-style responsive grid utilities

### Hugo Modules
~20+ modules are imported from `gethugothemes/hugo-modules` (see `config/_default/module.toml`). These provide: search, PWA, image processing, icons (Font Awesome), shortcodes (button, notice, accordion, tab, modal), cookie consent, announcements, SEO, and GTM. Module cache can be cleared with `npm run clear-modules` (calls `hugo mod clean`).

### Plugin System (CSS/JS)
External CSS and JS plugins are declared in `config/_default/params.en.toml` under `[[params.plugins.css]]` and `[[params.plugins.js]]`. Use `lazy = true` to defer loading. This is how Swiper, GLightbox, and other libraries are included.

### Section Content Pattern
Files in `content/[lang]/sections/` (e.g., `testimonial.md`, `call-to-action.md`) use `build.render = "never"` in front matter and are pulled into layouts via `site.GetPage "sections/call-to-action"`. These are data-like content files, not rendered pages.

### Dark Mode
Controlled by adding/removing `.dark` class on `<html>`. Colors for dark mode are defined in `data/theme.json` under `colors.darkmode`. Dark mode can be fully removed via `npm run remove-darkmode`.

## Content Front Matter

```yaml
---
title: "Post Title"
meta_title: ""          # Optional SEO override
description: ""
date: 2025-01-01T00:00:00Z
image: "/images/feature.png"
categories: ["Category"]
author: "Author Name"
tags: ["tag"]
draft: false
---
```

## Deployment

- **Netlify**: `netlify.toml` — runs `yarn project-setup; yarn build`
- **GitHub Actions**: `.github/workflows/main.yml` — deploys to GitHub Pages on push to `main`
- **Vercel/AWS Amplify**: config files at root handle those platforms

## Key Files

| File | Purpose |
|------|---------|
| `hugo.toml` | Main Hugo config (baseURL, theme, modules, markup) |
| `data/theme.json` | Design tokens (colors, fonts) |
| `data/social.json` | Social media links |
| `config/_default/params.*.toml` | Site parameters per language |
| `config/_default/module.toml` | Hugo module imports |
| `tailwind-plugin/tw-theme.js` | Reads theme.json → Tailwind tokens |
| `themes/hugoplate/` | Theme layouts, assets, and static files |
