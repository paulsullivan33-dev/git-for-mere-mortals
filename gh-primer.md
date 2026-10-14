# GitHub CLI Primer (gh) — the GitHub website in your terminal

`gh` is GitHub's official command-line tool. If the website is a
building full of rooms (pull requests, issues, Actions runs, releases),
`gh` is a set of doors into those rooms that you can open without
leaving your terminal.

The big idea first: **git and `gh` do different jobs.**

- **git** moves your save points around — between your machine and
  GitHub. (`git push`, `git pull` — see the git primer.)
- **`gh`** does the *GitHub* chores — the things you'd otherwise do by
  clicking around the website: opening pull requests, reading issues,
  watching Actions runs, making releases.

You still need git. `gh` doesn't replace it; it replaces the *browser
tab*. The daily rhythm becomes: do the file work with git, do the
GitHub paperwork with `gh`, never open the website unless you want to.

## What this guide covers

Installing `gh`, logging in, and the commands you'll actually use:
repos, issues, pull requests, Actions runs, and the `gh api` escape
hatch for everything else.

## What it doesn't cover

Writing Actions workflows (see the GitHub Actions primer), the GitOps
vocabulary (see that primer), or the GitHub REST API in depth — `gh
api` gets you started, and the API docs take it from there.

## Installing it

One command per platform, from GitHub's install docs at
[cli.github.com](https://cli.github.com):

```bash
# Windows (in PowerShell)
winget install --id GitHub.cli

# macOS
brew install gh

# Debian / Ubuntu: add GitHub's apt repository first
# (full steps at cli.github.com), then:
sudo apt install gh

# Fedora / Red Hat
sudo dnf install gh
```

I didn't run these installers here (`gh` was already on this machine),
so if your platform's package manager complains, cli.github.com is the
source of truth.

**Check:** (this one I did run)

```bash
gh --version
```

```
gh version 2.102.0 (2026-09-30)
https://github.com/cli/cli/releases/tag/v2.102.0
```

If that prints a version, you're installed.

## Logging in: `gh auth login`

`gh` needs proof that it's you before it can touch your GitHub account.
That proof is a **token** — a long secret string GitHub issues instead
of your password. You never type it by hand:

```bash
gh auth login
```

It asks a few questions (GitHub.com or a company server? HTTPS or SSH
for git?), then opens your browser with a one-time code. You approve
it there, and `gh` stores the token in your operating system's
credential vault — Windows Credential Manager, macOS Keychain, or the
Linux secret store. Not in a text file. You do this once per machine.

In plain language: it's like showing your ID at the door once and
getting a wristband. The wristband (token) gets you back in; the ID
(password) stays in your wallet.

```bash
gh auth status       # "am I logged in, and as who?"
```

**Check:** (run this yourself after logging in)

```bash
gh auth status
```

```
github.com
  ✓ Logged in to github.com account your-username (keyring)
```

Everything from here on needs that login. Nothing below was run
against a real account in this guide — the outputs marked
"yours will differ" are shapes, not transcripts. Run them yourself
after `gh auth login`.

### A note on tokens

If a script or a second machine needs access without the browser
dance, `gh auth login --with-token` reads a token from standard input
instead of opening a browser. Tokens are secrets — treat one like a
password: never paste it into chat, never commit it to a repo.

## Repos: looking and cloning

```bash
gh repo view                     # README and description of this folder's repo
gh repo view owner/repo          # any repo, by name
gh repo view --web               # open this repo in the browser instead
gh repo clone owner/repo         # git clone, but you can skip the URL
```

`gh repo view` is the fastest way to answer "what is this repo?"
without a browser tab. `gh repo clone` accepts extra `git clone`
flags after a `--` if you ever need them.

## Issues: the to-do list

Issues are GitHub's to-do items — bug reports, feature requests,
questions. Every repo has its own list.

```bash
gh issue list                    # open issues in this repo
gh issue view 12                 # read issue #12 in full
gh issue view 12 --web           # ...or open it in the browser
gh issue create --title "Login fails on expired sessions" --body "Steps to reproduce: ..."
```

Without `--title` and `--body`, `gh issue create` asks you
interactively. The `--web` flag is the escape hatch on every `view`
command: start in the terminal, jump to the browser when you want
pictures and formatting.

## Pull requests: the main event

A **pull request** (PR) is a proposal: "here are my save points, please
fold them into the official version." It's where code review happens.
This is where `gh` earns its keep.

```bash
gh pr status                     # PRs relevant to you, across states
gh pr list                       # PRs in this repo
gh pr view 42                    # read PR #42: title, body, checks, reviews
gh pr checkout 42                # download PR #42's branch and switch to it
```

`gh pr checkout 42` is the quiet superstar: a teammate's PR becomes a
local branch on your machine. You can run it, test it, poke at it —
then `git checkout main` to go home. No forking, no URL copying.

Creating one, from your feature branch:

```bash
gh pr create --fill              # title and body from your commit messages
gh pr create --title "Fix login bug" --body "Expired sessions now redirect to login."
gh pr create --draft             # open it as a draft (not ready for review)
```

`--fill` is the lazy-and-good option: if your commit messages are
clear ("Fix login bug on expired sessions"), they become the PR title
and body. `--title` and `--body` override whatever `--fill` guessed.

Merging, when the reviews are done:

```bash
gh pr merge 42 --squash --delete-branch
```

The three merge flavors, matching the buttons on the website:

| Flag | What it does | When |
|---|---|---|
| `--merge` | Keeps every commit, adds a merge commit | Preserving full history matters |
| `--squash` | Squashes your branch into one commit | The usual choice: one tidy save point |
| `--rebase` | Replays your commits onto the target branch | Linear history, no merge commits |

`--delete-branch` cleans up the feature branch afterwards, and
`--auto` waits for required checks to pass before merging. These all
mirror the website's merge buttons exactly — `gh` just skips the
clicking.

## Actions runs: watching the robots

Pairs with the GitHub Actions primer: after you push, the workflow
runs. Watch it from here:

```bash
gh run list                      # recent workflow runs
gh run watch                     # watch the latest run until it finishes
gh run watch --exit-status        # exit nonzero if the run fails (for scripts)
gh run view <run-id>             # details of one run
```

`gh run watch` is the terminal equivalent of staring at the Actions
tab — except it tells you when it's done so you don't have to stare.

## `gh api`: the escape hatch for everything else

The website does hundreds of things `gh` has no dedicated command
for. `gh api` covers all of them: it makes an authenticated GitHub
API request and prints the result. If the website can do it, `gh api`
can do it.

```bash
gh api repos/{owner}/{repo} --jq '.description'
```

(The `{owner}/{repo}` is a placeholder — put a real repo there.)

- `-X` / `--method` picks the HTTP verb (default `GET`; use `POST`
  to create things, `DELETE` to remove them).
- `-F` / `--field key=value` adds parameters.
- `--paginate` fetches all pages, not just the first.
- `-q` / `--jq` filters the JSON with jq syntax, so you get one
  value instead of a wall of text.

You don't need to learn the whole API to use this. Find the endpoint
in GitHub's REST API docs, translate the example into `gh api`, and
you're scripting anything.

## The mental model: git vs `gh` vs the website

| Tool | Its job | Example moment |
|---|---|---|
| `git` | Moves save points between your machine and GitHub | `git push` after committing |
| `gh` | Does GitHub chores from the terminal | `gh pr create` instead of clicking "New pull request" |
| Website | The visual version of `gh`'s chores | Reading a long PR discussion with screenshots |

Rule of thumb: **files → git, paperwork → `gh`**. If you're typing a
GitHub URL into the browser more than twice a day, there's probably a
`gh` command for it — check `gh <thing> --help`.

## Cookbook: the daily loop

```bash
# Morning: what's going on?
gh pr status                     # anything waiting on me?
gh run watch                     # is my workflow green yet?

# Teammate asked for a review:
gh pr view 42                    # read it
gh pr checkout 42                # try it locally
# ... test, then back:
git checkout main

# My turn to propose:
git checkout -b fix-login-bug
# ... work, commit ...
git push -u origin fix-login-bug
gh pr create --fill
```

## Troubleshooting

**"`gh` says I'm not authenticated"** (or every command fails with
an auth error):
```bash
gh auth login
```
Tokens expire or get revoked; re-running login fixes it.

**"It's acting on the wrong repo."** `gh` figures out which repo you
mean from the folder you're standing in. `cd` into the right clone,
or name the repo explicitly — most commands take `-R owner/repo`:
```bash
gh pr list -R owner/repo
gh issue view 12 -R owner/repo
```

**"I want the website for this one."** Every `view` command takes
`--web`. The terminal and the browser aren't rivals; use both.

**"The command I want doesn't exist."** That's what `gh api` is for
(see above). And `gh help` lists everything, grouped by topic.

## Words to know

Every term in this guide — token, scope, issue, pull request,
workflow run, release — is collected in the
[Glossary](glossary.md), in plain words.

## Validation notes

Written against `gh` 2.102.0. Every command and flag in this guide
was checked against the real `gh <command> --help` output on this
machine — no subcommand or flag is documented from memory. Anything
that needs a GitHub login (`auth status` output, issue/PR/run
listings) is marked as run-it-yourself; no API output was fabricated.
Install one-liners are from GitHub's install docs at cli.github.com
and were not executed here.
