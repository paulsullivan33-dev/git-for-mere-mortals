# Git Primer — a plain-language daily reference

Git feels confusing because tutorials start with commands. Start with the
*idea* instead: **git is a save-point system for your files**, like in a
video game. You play (edit files), you hit a save point (`commit`), and
you can always go back to any save point later. Everything else is
details.

## The big idea: three places

Your files live in three places. Every git command just moves stuff
between them:

1. **Working files** — the actual files on your disk, where you edit.
2. **Staging area** — a loading dock. You put files here when they're
   ready to be saved. (`git add`)
3. **Repository** — the save points themselves. (`git commit`)

And then there's a fourth place that matters at work:

4. **Remote** — the shared copy on GitHub. Your save points get
   *uploaded* there (`git push`) and other people's save points get
   *downloaded* from there (`git pull`).

That's the whole model. If a command confuses you, ask: "which two
places is this moving things between?"

## One-time setup

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Do this once per machine. It just labels your save points so people
know who made them.

## The 7 commands you'll use every day

| Command | Plain meaning | When |
|---|---|---|
| `git status` | "What's going on?" | All the time. Run it whenever you're unsure. |
| `git pull` | Download teammates' save points | Before you start work, and before you push |
| `git add <file>` | Put a file on the loading dock | When a file is ready to save |
| `git commit -m "message"` | Create a save point | When something works — commit early, commit often |
| `git push` | Upload your save points to GitHub | When you want others to see your work |
| `git log --oneline` | List recent save points | "What happened lately?" |
| `git diff` | Show what changed but isn't saved yet | Before you commit, to double-check |

The daily rhythm is: `pull` → work → `add` → `commit` → `push`.

## Branches: the slightly bigger idea

A **branch** is just a label stuck on a save point. `main` is the
official label — the version everyone agrees is "the real thing."

When you start new work, you make your own label (branch), work there,
and later fold it back into `main`. This keeps unfinished work from
touching the official version.

```bash
git checkout -b my-feature     # make a branch AND switch to it
git branch                     # list your branches (* = where you are)
git checkout main              # switch back to main
git merge my-feature           # fold my-feature's save points into main
git branch -d my-feature       # delete the branch label (save points stay)
```

(`git switch` does what `git checkout` does for branches, with a
clearer name. Either works.)

**Rule of thumb:** `main` is shared and sacred. Your branch is your
scratch pad. Merge when the work is done and tested.

## Cookbook: common situations

### Starting new work
```bash
git checkout main
git pull                       # get the latest official version
git checkout -b fix-login-bug  # your own branch, named after the work
# ... edit files ...
git add .
git commit -m "Fix login bug on expired sessions"
git push -u origin fix-login-bug   # -u links your branch to GitHub, first time only
```

### Saving your work (the normal loop)
```bash
git status                     # check what's changed
git diff                       # review the changes
git add <files>                # stage them (or: git add . for everything)
git commit -m "Short, clear message"
```

Write commit messages like a label: "Fix X", "Add Y", "Remove Z".

### I messed up a file — undo my edits (not saved yet)
```bash
git restore <file>             # throw away my edits to this file
```
Careful: this deletes your edits permanently. Only for changes you
truly don't want.

### I staged something I didn't mean to
```bash
git restore --staged <file>    # un-stage it; your edits stay in the file
```

### I committed but want to redo it (not pushed yet)
```bash
git reset --soft HEAD~1        # undo the last save point, keep my edits
```
Then fix things up and commit again.

### I need to switch tasks but I'm mid-work — shelve it
```bash
git stash                      # pack up my unfinished changes
# ... do the other thing ...
git stash pop                  # unpack them again
```

### Getting a coworker's branch
```bash
git fetch                      # learn about new branches on GitHub
git checkout their-branch-name # switch to it (git grabs it automatically)
```

### Merge conflict — git can't combine our changes
This happens when you and someone else edited the *same lines* of the
*same file*. Git stops and asks you to decide. It's normal, not an
error.

1. Open the file. You'll see markers:
   ```
   <<<<<<< HEAD
   your version of the lines
   =======
   their version of the lines
   >>>>>>> their-branch
   ```
2. Edit the file to what it *should* say. Delete all the markers.
3. Save, then:
   ```bash
   git add <file>
   git commit -m "Resolve merge conflict in <file>"
   ```

### Oops, I committed straight to main
```bash
git checkout -b my-feature     # move the commit onto a new branch
git checkout main
git reset --hard origin/main   # put main back where GitHub has it
```
(The `--hard` here is safe *only* because your commit is now safe on
the new branch.)

### I merged a PR on GitHub and deleted the branch, but my computer still shows it
```bash
git checkout main
git fetch --prune          # refresh your list of GitHub's branches
git branch -d branch-name  # delete your local copy of the branch
```
Why: your computer keeps its own list of GitHub's branches, and it
doesn't learn about deletions until you fetch with `--prune`. VS Code
reads that stale list, which is why it shows errors.

Make it automatic so you never think about it again:
```bash
git config --global fetch.prune true
```

## Golden rules

1. **Pull before you push.** Gets you the latest and avoids most conflicts.
2. **Commit early, commit often.** Small save points are easy to
   understand and easy to undo.
3. **Never rewrite shared history.** Don't use `--force` on `main` or
   anyone else's branch. (On *your own* branch that nobody else uses,
   it's fine.)
4. **main always works.** If it's broken, fixing it is the top priority.
5. **When in doubt, `git status`.** It tells you where you are and often
   suggests the next command.

## Glossary in plain words

| Word | Means |
|---|---|
| **repository** (repo) | The project folder plus all its save points |
| **commit** | One save point — a snapshot of your files plus a message |
| **branch** | A movable label on a save point; `main` is the official one |
| **remote** | The shared copy (GitHub). Nicknamed `origin` by default |
| **clone** | Download a repo for the first time (`git clone <url>`) |
| **merge** | Fold one branch's save points into another |
| **conflict** | Git can't auto-combine two edits; a human decides |
| **stash** | Temporarily shelve unfinished changes |
| **fetch** | Ask GitHub "what's new?" without downloading files yet |
| **HEAD** | "Where I am right now" (the current save point) |
