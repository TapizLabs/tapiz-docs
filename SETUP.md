# Setting Up the tapiz-docs GitHub Repository

This file explains how to publish this documentation folder as a GitHub repository.

---

## Option A — Using the GitHub CLI

```bash
# 1. Install GitHub CLI if you haven't: https://cli.github.com/
gh auth login

# 2. Navigate to this folder
cd tapiz-docs

# 3. Init git, commit, and push
git init
git add .
git commit -m "docs: initial documentation for tapiz-rest-api and tapiz-reactjs-ui"

gh repo create tapiz-docs --public --source=. --remote=origin --push
```

---

## Option B — Manual (GitHub website)

1. Go to https://github.com/new
2. Repository name: `tapiz-docs`
3. Set visibility (Public or Private)
4. **Do not** initialize with a README (you already have one)
5. Click **Create repository**
6. Follow the "push an existing repository" instructions shown:

```bash
cd tapiz-docs
git init
git add .
git commit -m "docs: initial documentation"
git branch -M main
git remote add origin https://github.com/<your-username>/tapiz-docs.git
git push -u origin main
```

---

## Recommended Repository Settings

After creating:

- **About** → add description: *"Documentation for the Tapiz attendance tracking platform"*
- **Topics** → `documentation`, `typescript`, `react`, `hono`, `postgresql`
- **Pages** (optional) → enable GitHub Pages on `main` branch to render the Markdown as a website

---

## Keeping Docs Updated

Whenever you make significant changes to either project, update the relevant file:

| Changed | Update |
|---------|--------|
| New API endpoint | `api/API_REFERENCE.md` |
| New DB table | `api/DATABASE.md` |
| Architecture change | `api/ARCHITECTURE.md` |
| New UI page or feature | `ui/README.md` |
| New env variable | `api/README.md` or `ui/README.md` |
