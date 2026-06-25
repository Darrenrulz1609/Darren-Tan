---
description: Push to GitHub, deploy GitHub Pages via Actions, create/update README, set repo About + live URL, and security-scan for secrets before publishing.
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# /github-push-by-darrenrulz

Publish this project (`Sterling & Vale Advisory`, a single-file `index.html` static site) to GitHub
and deploy it as a live GitHub Pages site. Do the steps **in order** and stop to report if any step
fails or needs the user's input (e.g. authentication, choosing a repo name).

## 0. Pre-flight

1. Confirm `git` and the GitHub CLI (`gh`) are available: run `git --version` and `gh --version`.
   - If `gh` is missing, tell the user to install it (https://cli.github.com/) and stop.
2. Confirm the user is authenticated: `gh auth status`.
   - If not authenticated, tell them to run `gh auth login` and stop.
3. Check whether a remote already exists: `git remote -v`.
   - If an `origin` remote exists, reuse it (this is an update/re-push).
   - If not, this is a first-time publish — you'll create the repo in step 3.

## 1. SECURITY SCAN (do this BEFORE anything is pushed)

This is the most important step. **Nothing leaves the machine until this passes.** Scan the entire
working tree (everything that would be committed) for sensitive information:

- API keys / tokens (e.g. `AKIA…`, `ghp_…`, `sk-…`, `xox[baprs]-…`, Google `AIza…`, generic
  `*_API_KEY`, `*_SECRET`, `*_TOKEN`, bearer tokens).
- Private keys / certs (`-----BEGIN … PRIVATE KEY-----`, `.pem`, `.p12`, `.pfx`, `.key`).
- Passwords / connection strings / DSNs with embedded credentials.
- `.env` files, credential JSON, cloud service-account files.
- Personal data that should not be public.

How to scan:
1. Use `Grep` across the repo for the patterns above (case-insensitive where appropriate).
2. Inspect `index.html` specifically — the enquiry form's `ENDPOINT` constant contains an **email
   address** (currently `darren.tan@redbeaconam.com`). A destination email in a public static
   FormSubmit form is expected/acceptable, but **call it out explicitly** so the user confirms they
   are OK publishing that address. Do not treat it as a blocker, but require an acknowledgement.
3. Check `git status` / staged files for anything that looks secret (`.env`, key files, dumps).

If any real secret (key/token/password/private key) is found:
- **STOP.** Do not push. List exactly what and where, and recommend removing it (and rotating the
  secret if it was ever committed). Wait for the user.

If only the form email is found, note it, get the user's go-ahead, then continue.

## 2. README

Create or update `README.md` at the repo root. It should include:
- Project title (**Sterling & Vale Advisory**) and a one-line description (single-page marketing
  site for a fictional investment advisory firm; vanilla HTML/CSS/JS, no build step).
- A **Live site** link — use the GitHub Pages URL (fill in after you know the repo:
  `https://<user>.github.io/<repo>/`). If updating later, ensure this URL is correct.
- How to run locally (`open index.html`, or `python -m http.server` then `localhost:8000`).
- Brief tech notes: single self-contained `index.html`, Google Fonts via `<link>`, FormSubmit AJAX
  enquiry form.
- A short "Deployment" note: auto-deployed to GitHub Pages via GitHub Actions on push to `main`.

Keep it concise and accurate to the actual project — do not invent features.

## 3. Repo + GitHub Pages deploy workflow

1. If there is no `origin` remote yet, create the repo and push:
   - Ask the user for a repo name if not obvious (suggest `sterling-vale-advisory`), and whether it
     should be **public** (required for free GitHub Pages) or private.
   - `git init` (if not already a repo), stage, commit, then:
     `gh repo create <name> --public --source=. --remote=origin --push`
2. Add a GitHub Actions workflow to deploy Pages. Create
   `.github/workflows/deploy-pages.yml` with a standard static-site Pages deploy (upload the repo
   root as the artifact, deploy with `actions/deploy-pages`). It must:
   - Trigger on `push` to `main` (and `workflow_dispatch`).
   - Set `permissions: pages: write, id-token: write, contents: read`.
   - Use `actions/configure-pages`, `actions/upload-pages-artifact` (path `.`), and
     `actions/deploy-pages`.
3. Enable Pages to build from GitHub Actions:
   `gh api -X POST repos/{owner}/{repo}/pages -f build_type=workflow` (ignore "already exists"
   errors; if Pages already configured, ensure `build_type` is `workflow`).
4. Commit the workflow + README, then push to `main`:
   `git add -A && git commit -m "Add Pages deploy workflow, README, and publish" && git push -u origin main`
5. Watch the deployment: `gh run list --limit 1` then `gh run watch <run-id>` (or poll
   `gh run list`). Report success/failure. The live URL is `https://<owner>.github.io/<repo>/`.

## 4. Repo "About" (description + live site)

Once the site is live, update the repository's About section:
- Set the description and the homepage URL to the live Pages URL:
  `gh repo edit <owner>/<repo> --description "Single-page marketing site for Sterling & Vale Advisory (fictional investment advisory firm) — vanilla HTML/CSS/JS." --homepage "https://<owner>.github.io/<repo>/"`
- Optionally add topics: `gh repo edit <owner>/<repo> --add-topic html,css,javascript,github-pages,static-site`.

## 5. Final report

Summarize for the user:
- ✅ Security scan result (and the form-email acknowledgement).
- ✅ Repo URL.
- ✅ Live GitHub Pages URL (and Actions run status).
- ✅ README + About updated.
- Any follow-ups (e.g. FormSubmit activation: the first enquiry submission triggers a one-time
  confirmation email that must be clicked before the form delivers anything).

## Notes / guardrails

- Never push if the security scan finds a real secret — stop and report.
- Don't fabricate a live URL; derive it from the actual owner/repo.
- Keep the site vanilla — do not add frameworks or a build step just to deploy.
- If `$ARGUMENTS` is provided, treat it as the desired repo name and/or visibility.
