---
title: "Git: push the same code to GitHub and GitLab from a single local repo"
date: 2026-01-12T00:00:00+01:00
draft: false
tags: ["git", "github", "gitlab", "devops", "workflow"]
categories: ["guides"]
description: "Process to configure a local repo with a remote called 'remote' that can push to GitHub, GitLab, or both, and to sync branches."
featureimage: "/images/posts/2026-01-12-una-fuente-de-verdad/feature.svg"
---

# Git: push the same code to GitHub and GitLab from a single local repo (remote called `remote`)

![Illustration of one repo feeding two remotes](/images/posts/2026-01-12-una-fuente-de-verdad/feature.svg)

Do you want **a single local repo** and to push to **GitHub and GitLab** without breaking anything?  
Here’s a quick guide with zero drama. 😎

---

## 🎯 Goal

Have **one local repository** and keep **two remote repositories with the same code**:

- **Repo 1:** GitHub  
- **Repo 2:** GitLab  
- The “main” remote in your local repo might be called **`origin`** (not `remote`).

Also:
- Know **when and how** to push **only to GitHub**, **only to GitLab**, or **to both**.
- Learn how to **sync branches** (and optionally tags) so both remotes stay aligned.

---

## 🗺️ Conventions used in this guide

Replace these URLs with yours:

- GitHub: `https://github.com/<org>/<repo>.git`
- GitLab: `https://gitlab.com/<org>/<repo>.git`

Main branch: `main` (if you use `master`, replace `main` with `master` in all commands).

---

## ✅ 1) Initial state (or verification)

In your local repo, list remotes:

```bash
git remote -v
```

If a remote called `remote` already exists, you’ll see something like:

```bash
remote  https://github.com/<org>/<repo>.git (fetch)
remote  https://github.com/<org>/<repo>.git (push)
```

If it doesn’t exist, you’ll create it in the next step. Easy.

---

## 🔗 2) Configure remotes for GitHub + GitLab

### Case A: You don’t have the `remote` remote yet

Configure it pointing to GitHub first (or GitLab if you prefer):

```bash
git remote add remote https://github.com/<org>/<repo>.git
```

Now add explicit remotes so you can separate pushes when needed:

```bash
git remote add github https://github.com/<org>/<repo>.git
git remote add gitlab https://gitlab.com/<org>/<repo>.git
```

Yes: you’ll have three names (`remote`, `github`, `gitlab`) on purpose.  
`github` and `gitlab` are for selective pushes, and `remote` is the alias to push to both.

Verify:

```bash
git remote -v
```

### Case B: You already have `remote` and want to add GitLab

First create explicit remotes:

```bash
git remote add github https://github.com/<org>/<repo>.git
git remote add gitlab https://gitlab.com/<org>/<repo>.git
```

Then we’ll adjust `remote` so it can push to both (below).

---

## 🚀 3) Split the push: repo1, repo2, and both

Here’s the important part. We’ll go through the three modes:

### Mode 1 — Push ONLY to GitHub (repo1)

```bash
git push github main
```

When to use it:
- If GitHub is the “main” repo (PRs, checks, releases).  
- If you want to test on one remote and not touch the other yet.

### Mode 2 — Push ONLY to GitLab (repo2)

```bash
git push gitlab main
```

When to use it:
- If GitLab is used for internal pipelines.
- If you want to update the mirror without affecting GitHub.

### Mode 3 — Push to BOTH with one command (using `remote`)

We’ll configure `remote` to push to two URLs.

#### 3.1 Configure `remote` with multiple push URLs

First make sure you have `remote` created:

```bash
git remote add remote https://github.com/<org>/<repo>.git
```

Now add two push URLs to `remote`:

```bash
git remote set-url --add --push remote https://github.com/<org>/<repo>.git
git remote set-url --add --push remote https://gitlab.com/<org>/<repo>.git
```

Optional (and recommended): set the fetch URL of `remote` to a single one (for example GitHub):

```bash
git remote set-url remote https://github.com/<org>/<repo>.git
```

Verify:

```bash
git remote -v
```

You should see `remote` with one fetch and two pushes (sometimes listed twice for push).

#### 3.2 Push to both

```bash
git push remote main
```

When to use it:
- When GitHub and GitLab must always be in sync at the same time.
- When you’re running an operational “mirror” (without cloning duplicate repos).

> Note: If one push fails (permissions, branch protection, etc.), Git will tell you.  
> The other may have gone through, so check the command output. 👀

---

## 🔁 4) Sync branches (so GitHub and GitLab stay identical)

True sync boils down to this: push the same branches to both remotes.

### 4.1 Sync a specific branch (e.g., `main`)

To GitHub:

```bash
git push github main
```

To GitLab:

```bash
git push gitlab main
```

To both (if you configured `remote` with two push URLs):

```bash
git push remote main
```

### 4.2 Sync ALL local branches to the remote

To GitHub:

```bash
git push github --all
```

To GitLab:

```bash
git push gitlab --all
```

---

## ✅ Mini visual checklist

- [ ] I have `remote`, `github`, and `gitlab` created.  
- [ ] I know when to push to one or the other.  
- [ ] I can push to both with `git push remote main`.  
- [ ] My branches are synced on both remotes.

To both:

```bash
git push remote --all
```
What does `--all` do?

Pushes all local branches (not tags).

Very useful when you’re setting up the mirror for the first time.

4.3 Sync tags (recommended if you do releases)  
Tags are NOT included with `--all`. For tags:

To GitHub:

```bash
git push github --tags
```
To GitLab:

```bash
git push gitlab --tags
```
To both:

```bash
git push remote --tags
```
4.4 Typical full sync (branches + tags)  
To leave both remotes identical, you usually do:

```bash
git push remote --all
git push remote --tags
```

5) Best practices and typical gotchas  
Branch protection  
If main is protected (very common), you may find:

GitHub only allows push via PR, and GitLab too.

Then direct push will fail on one or both.

Solution: push a feature branch and open a PR/MR on each platform, or decide one as the “source of truth.”

Source of truth

Decide whether:

GitHub is the source of truth and GitLab is a mirror, or vice versa.

Avoid having divergent changes in both (for example, independent merges on each).
If you do, you’ll end up with conflicts or different histories.

Pushing “to both” is powerful… and dangerous  
Use `remote` (double push) when you’re clear that:

the same rules (protections) apply,

and you want permanent sync.

If not, use separate pushes (`github` / `gitlab`) to control timing.

6) Quick checklist (operational procedure)  
Setup (one time)  
Create github and gitlab remotes

Configure remote to push to both

Daily operation  
Only GitHub:

```bash
git push github <branch>

```
Only GitLab:

```bash
git push gitlab <branch>

```
Both:

```bash
git push remote <branch>

```
Sync everything

```bash
git push remote --all
git push remote --tags
```

7) Final verification: check that everything is aligned  
List the remote branches your local sees:

```bash
git fetch --all --prune
git branch -r
```

You should see something like:

```bash
github/main
gitlab/main
...
```

If you see differences (branches that exist on one remote and not the other), repeat `push --all` to the missing one.

Conclusion  
With this setup:

github and gitlab give you fine-grained control (selective push).

remote lets you do a “single push” to both when you’re sure they must stay synced.

For full sync: `--all` + `--tags`.
