# GitOps Vocabulary — plain-language companion

## The big idea (read this first)

GitOps is a way of running systems with one rule: **the git repo is the
truth about what should be running.** You want to change the system? You
change files in git — not by clicking around a console, not by SSHing
into a server and typing commands. A robot watches the repo, and whenever
the repo changes, the robot makes reality match.

Think of it like this: the repo is the recipe card on the fridge. Nobody
cooks from memory and nobody improvises — if it's not on the card, it
doesn't happen. The robot is the cook that follows the card exactly.

## The vocabulary

| Word | Plain meaning |
|---|---|
| **Repository** | The source of truth. It describes what *should* exist: apps, settings, how many copies of each thing should run. |
| **Commit** | A proposed change to the system, saved as a git save point. "Change the website to 3 copies." |
| **Pull request (PR)** | "Please apply this change." A commit put up for review before it becomes official. Teammates read it, comment, approve. |
| **Merge** | The change is approved and becomes the truth. In GitOps, **merging often IS deploying** — the robot picks it up from here. |
| **Pipeline** | The robot assembly line. A series of automatic steps (often a GitHub Actions workflow) that checks and then applies your change. |
| **CI** (Continuous Integration) | The "checking" half of the pipeline. Every PR gets automatically tested: does it build? Do the tests pass? Is the config valid? |
| **CD** (Continuous Deployment/Delivery) | The "applying" half. Takes the approved change and puts it on the real servers. *Deployment* = automatic; *Delivery* = one button-press away. People mix them up constantly — now you know. |
| **Declarative** | Describing what you **want**, not how to do it. "I want 3 copies running" (declarative) vs "start a copy, then start another, then another" (imperative). GitOps is declarative — the robot figures out the how. |
| **Desired state** | What git says should be running. |
| **Actual state** | What is *really* running right now. |
| **Reconciliation loop** | The robot's endless chore: compare desired state to actual state, and fix anything that doesn't match. Runs over and over, forever. |
| **Drift** | Reality wandered away from the recipe. Someone hand-edited a server "just this once." The next reconciliation loop either puts it back or raises an alarm — which is the whole point. |
| **Environment** | A separate copy of the system: **dev** (your playground), **staging** (dress rehearsal), **prod** (the real thing customers use). Often one branch or folder per environment. |
| **Promotion** | Moving a change up the ladder: dev → staging → prod. Usually done with a PR or a merge, so it's reviewed at each step. |
| **Rollback** | Undo. Something broke in prod? Go back to the earlier commit that worked. Because every change is a git save point, undoing is just pointing at the old one — the robot restores it. |
| **Artifact** | The built thing the pipeline produces: a container image, a package, a zip file. Code goes in, artifact comes out, artifact gets deployed. |
| **Tag / Release** | A labeled snapshot: "v1.2.3". Marks exactly what's in prod right now, so everyone can point at it and say "that one." |

## How it flows, end to end

Follow one change all the way through:

1. **You edit a file** in git: `replicas: 2` becomes `replicas: 3`.
   (One line. That's the whole "change request.")
2. **Commit, push to a branch, open a PR.** "Please run 3 copies of the
   website."
3. **CI robot checks it.** Tests pass, the config is valid YAML, nothing
   looks crazy. Green check on the PR.
4. **A teammate approves.** You click **Merge**. The change is now the
   official truth on `main`.
5. **CD robot notices** `main` changed. It takes the new recipe and
   applies it: a third copy starts up on the real servers.
6. **The reconciliation loop keeps watching.** Next week someone SSHes
   in and kills a copy "temporarily." The robot sees actual state (2)
   ≠ desired state (3) and starts a replacement — or pages someone,
   depending on setup.

Notice what never happened: nobody SSHed into prod to make the change.
The change went through git, review, robots — and there's a permanent
record of who asked for it, who approved it, and when it landed.

## Why teams like it

- **History.** Every change is a commit: who, what, when, why. "Who
  turned this off?" has an answer.
- **Review.** Nothing reaches prod without a PR. No more mystery
  midnight edits.
- **Undo.** Rollback is "go back to that commit." Compare that to
  reverse-engineering someone's hand-typed server commands at 2 AM.
- **One process for everything.** App code and infrastructure go
  through the same PR → review → robot flow. Less to learn, fewer
  special cases.

## The gotchas (plain words)

- **Secrets don't go in git.** Passwords and tokens live in the secrets
  system (like the Secrets section of the Actions primer), never in a
  committed file. A secret in git history is compromised the second it's
  pushed.
- **The repo must REALLY be the only way in.** If people can still SSH
  in and hand-edit prod, drift wins and the "source of truth" is a
  polite fiction. GitOps only works if the back door is locked.
- **Review PRs like they matter — because merging IS deploying.** That
  "looks fine" click can restart production. Read the diff.
