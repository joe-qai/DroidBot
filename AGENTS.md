# AGENTS.md - DroidBot Frontend

## Project Type
Static HTML site (frontend demo/docs). The Python backend is in a separate repo: `https://github.com/joe-qai/DroidBot.git`

## Deployment
- Push to `main` branch triggers automatic deployment to GitHub Pages via `.github/workflows/static.yml`
- Manual trigger available via Actions tab

## Local Preview
Open any `.html` file directly in browser, or use a static server:
```powershell
python -m http.server 8000
```

## Key Files
- `index.html` - Main landing page
- `demo.html` - Interactive demo dashboard
- `devices.html`, `execution.html`, `reports.html`, etc. - Other demo pages

## Development Notes
- No build process, no package manager, no Python in this repo
- All HTML files are self-contained with inline CSS
- No tests, linting, or typechecking in this repo

## Related Documentation
- `Android兼容性测试Agent技术架构方案.md` - Architecture design doc
- `兼容性测试自动化Agent方案.md` - Additional technical docs