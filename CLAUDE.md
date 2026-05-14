# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DroidBot is an AI-driven Android compatibility testing platform. **This repo contains only the frontend** — static HTML demo/prototype pages. The Python backend (FastAPI + uiautomator2) lives in a separate repo: `https://github.com/joe-qai/DroidBot.git`.

## Architecture & Design Patterns

- **Static HTML site**: No build process, no package manager, no bundler. Every `.html` file is self-contained with inline CSS and JavaScript.
- **Dark theme UI**: All pages use CSS custom properties (`:root` variables) for colors, gradients, and spacing. Two theme variants exist — `index.html` uses a cyan/blue accent palette (`#00d4ff` / `#0099ff`), while `demo.html` and functional pages use a purple gradient palette (`#667eea` / `#764ba2`).
- **Layout pattern**: Functional pages (`demo.html`, `devices.html`, etc.) share a sidebar+main content layout with a fixed 260px sidebar containing navigation. `index.html` is a standalone landing/marketing page.
- **Language**: All UI text is in Chinese (zh-CN). Technical docs (`Android兼容性测试Agent技术架构方案.md`, `兼容性测试自动化Agent方案.md`) are also in Chinese.
- **No shared CSS/JS**: Each page duplicates its own styles and scripts. There is no shared stylesheet or JS module file.

## Commands

### Local Preview
```powershell
python -m http.server 8000
```
Then open `http://localhost:8000/index.html` in a browser. You can also open any `.html` file directly.

### Deployment
Push to `main` triggers automatic GitHub Pages deployment via `.github/workflows/static.yml`. Manual deploy available from the Actions tab.

## Key Files

| File | Purpose |
|------|---------|
| `index.html` | Landing/marketing page (standalone, no sidebar) |
| `demo.html` | Dashboard hub with sidebar navigation to all other pages |
| `devices.html` | Connected device list and status |
| `new-test.html` | Natural language test creation interface |
| `ai-script.html` | AI-generated script preview |
| `execution.html` | Test execution monitoring |
| `reports.html` | Test report gallery |
| `report-detail.html` | Detailed report view |
| `apk-manager.html` | APK upload and management |
| `settings.html` | LLM provider and system configuration |

## Development Constraints

- No tests, linting, or type checking exist in this repo.
- No Python code in this repo — it's purely frontend HTML.
- When modifying pages, all CSS and JS must remain inline within each HTML file since there's no asset pipeline.
- The two CSS variable palettes (cyan for `index.html`, purple for dashboard pages) are intentional design choices — don't unify them without checking both look correct.