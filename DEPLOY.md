# Deployment & Update Procedure

This project builds the Jekyll site using GitHub Actions and publishes the generated `_site` to the `gh-pages` branch. Follow these steps to ensure the site deploys correctly and to update the site safely.

## Overview
- The repository contains a workflow at `.github/workflows/jekyll-build-deploy.yml` that runs on `push` to `main` and the feature branch `enhance/header-typography` and on manual dispatch.
- The workflow builds the site using the repository `Gemfile` and deploys the site to the `gh-pages` branch using `peaceiris/actions-gh-pages`.

## Required Repository Settings
1. Enable workflow write permission:
   - Go to `Settings` → `Actions` → `General` → `Workflow permissions` and choose **Read and write**. Save the change.
2. GitHub Pages source:
   - After the workflow runs and publishes to `gh-pages`, go to `Settings` → `Pages` and set **Source** to the `gh-pages` branch (root). This will serve the content from `gh-pages`.

## How to update the site (normal flow)
1. Make content or layout changes on `main` (or a branch), then push or open a PR.
2. Merge the PR into `main` (or push to `main`). The workflow triggers and will:
   - Install dependencies from `Gemfile` (uses Bundler cache)
   - Run `bundle exec jekyll build` to generate `_site`
   - Deploy `_site` to `gh-pages` branch
3. After the workflow finishes, Pages (if set to `gh-pages`) will serve the updated site. Allow a minute or two for propagation.

## How to trigger a manual deploy
- From the Actions tab in GitHub, choose **Build and deploy Jekyll site** and click **Run workflow**, select the `main` branch or the branch you want to build.
- Or use GitHub CLI (if installed):
```bash
gh workflow run "Build and deploy Jekyll site" --repo sondosclick/sondos.click --ref main
```

## Using a Personal Access Token (optional)
If your organization or repository policies prevent the default `GITHUB_TOKEN` from pushing to `gh-pages`, create a PAT and store it as a repository secret:

1. Create a PAT (classic) with the `repo` (private) or `public_repo` (public) scope.
2. Add it to repository secrets: `Settings` → `Secrets and variables` → `Actions` → `New repository secret`.
   - Name the secret `GH_PAGES_PAT`.
3. The workflow can be updated to use `${{ secrets.GH_PAGES_PAT }}` for `github_token` in the `peaceiris/actions-gh-pages` step. If you want me to patch the workflow to prefer the PAT when present, I can do that.

## Local preview (recommended before pushing)
1. Install Ruby, Bundler and build tools (on openSUSE):
```bash
sudo zypper refresh
sudo zypper install -y gcc gcc-c++ make autoconf automake libtool pkg-config \
  ruby-devel zlib-devel openssl-devel libffi-devel
gem install bundler --no-document
```
2. Install gems and serve locally:
```bash
bundle config set --local path 'vendor/bundle'
bundle install --jobs=4 --retry=3
bundle exec jekyll serve --livereload
```
3. Open http://127.0.0.1:4000/ and verify the site.

## Troubleshooting
- If CSS or posts show 404s on the GitHub Pages site, ensure `_config.yml` has `baseurl: "/<repo-name>"` when hosting at `https://<user>.github.io/<repo-name>/`. In this repo it is set to `/sondos.click`.
- If the workflow fails while pushing to `gh-pages` with a 403, check `Workflow permissions` (must be **Read and write**) and branch protection rules for `gh-pages`.
- If eventmachine or other native gems fail to build locally, install system build dependencies (see `Local preview` section). Using the CI avoids local native build issues.

## Rollback
- You can revert the `gh-pages` branch to a previous commit via the UI or locally and then push to `gh-pages`. Example:
```bash
git fetch origin gh-pages
git checkout gh-pages
git reset --hard <previous-commit-sha>
git push --force origin gh-pages
```

---

If you want, I can also:
- Change the workflow to prefer a `GH_PAGES_PAT` secret when available (safer for restricted orgs).
- Update the workflow to trigger on merges only (instead of every push) or to publish only from `main`.
