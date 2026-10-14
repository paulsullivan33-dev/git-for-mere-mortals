# Git for Mere Mortals

Git is the save-point system underneath modern software work: every
change a snapshot, every snapshot labeled, whole histories you can
rewind. This guide collects four plain-language primers — the git
you'll use every day, the GitHub CLI, GitHub Actions, and the GitOps
vocabulary — written for people learning it for work, not for people
who already know it.

No jargon without an explanation. No "it's simple, just..." Every
concept starts with the *idea*, then the commands.

## What this guide covers

- [**Git primer**](git-primer.md) — the save-point mental model, setup, the 7 commands
  you'll use every day, branches, and a cookbook of real situations:
  undoing mistakes, merge conflicts, stashing, cleaning up branches
  deleted on GitHub
- [**GitHub CLI primer (gh)**](gh-primer.md) — the GitHub website in your terminal: repos,
  issues, pull requests, Actions runs, and the `gh api` escape hatch
- [**GitHub Actions primer**](github-actions-primer.md) — workflows, events, jobs, steps, secrets,
  and self-hosted runners on Linux and Windows, from trigger to
  green check
- [**GitOps vocabulary**](gitops-vocabulary.md) — what repo, PR, pipeline, CI/CD, drift, and
  rollback mean when "the repo is the truth about what should be
  running"

## What it doesn't cover

Deep GitHub Actions (expressions, environments, reusable workflows)
and the full GitOps practice — both have their own guides, linked
under "Go deeper" below. This guide is the working vocabulary and
daily reference, not the whole subject.

## How to use this guide

Start with the git primer, then pick up the others as needed. Every
git command in the primer was run against real git before shipping;
the GitHub CLI commands were checked against the real `gh`, with
login-required output clearly marked. The cookbook grows from real
messes — hit a confusing git situation at work? That's the next
entry.

## Chapters

1. [Git Primer](git-primer.md)
2. [GitHub CLI Primer (gh)](gh-primer.md)
3. [GitHub Actions Primer](github-actions-primer.md)
4. [GitOps Vocabulary](gitops-vocabulary.md)
5. [Glossary](glossary.md)

## Go deeper

- [GitHub Actions for Mere Mortals](https://paulsullivan33-dev.github.io/github-actions-for-mere-mortals/) —
  the full guide: expressions, contexts, environments, reusable
  workflows, OIDC, starter workflows
- [GitOps for Mere Mortals](https://paulsullivan33-dev.github.io/gitops-for-mere-mortals/) —
  the full guide: the practice behind the vocabulary

*Written to the [nine-point writing standard](criteria.md) shared by
every guide in this series.*
