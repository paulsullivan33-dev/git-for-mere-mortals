# GitHub Actions Primer — plain-language guide

## The big idea (read this first)

GitHub Actions is a **robot helper that does chores automatically** when
things happen in your repository. You push code → the robot runs your
tests. You open a pull request → the robot checks it. Every night at
2 AM → the robot builds a report.

You describe the chores in a recipe file. GitHub finds a computer to run
the recipe. The robot does the work and reports back with a green check
or a red X. That's the whole system.

## The vocabulary (six words)

| Word | Plain meaning |
|---|---|
| **Workflow** | The recipe. A YAML file in `.github/workflows/` that says "when X happens, do Y." |
| **Event** (trigger) | The doorbell. The thing that starts the robot: a push, a pull request, a schedule, a manual button. |
| **Job** | One chunk of work in the recipe. Each job runs on its own computer. Jobs run in parallel unless you say otherwise. |
| **Step** | One instruction inside a job. Steps run **in order**, top to bottom. |
| **Action** | A pre-made recipe card. Instead of writing "install Python" yourself, you use someone's tested card: `actions/setup-python@v5`. |
| **Runner** | The computer that does the work. Either GitHub's computer (hosted) or **your** computer (self-hosted). |

## How it works, end to end

Follow one push all the way through:

1. **You push** to `main`. That's the doorbell (`on: push`).
2. **GitHub looks** in `.github/workflows/` for recipes listening for
   a push. It finds yours and creates a **run** (one execution of the
   recipe).
3. **Each job gets a runner.** The recipe says `runs-on: ubuntu-latest`,
   so GitHub picks one of its Linux computers. (With a self-hosted
   runner, your own machine picks up the job instead.)
4. **The runner works through the steps** in order: download your code,
   install Python, run the tests. Output streams live to the run's page.
5. **GitHub reports back.** Green check on your commit if everything
   passed, red X if something failed. Click the X to read exactly which
   step broke and why.

## Your first workflow

Create `.github/workflows/tests.yml` in your repo:

```yaml
name: Tests                  # shows up as the run's title

on: [push]                   # the doorbell: run on every push

jobs:
  test:                      # one job, named "test"
    runs-on: ubuntu-latest   # use GitHub's Linux computer

    steps:                   # do these in order:
      - uses: actions/checkout@v4        # 1. download my code
      - uses: actions/setup-python@v5    # 2. install Python (pre-made card)
        with:
          python-version: '3.12'
      - run: pip install -r requirements.txt   # 3. install my libraries
      - run: python -m pytest                  # 4. run the tests
```

Push that file, then go to the **Actions** tab in your repo. You'll see
the run, each step, and its log. Break a test on purpose once — watching
a run go red and reading the log is the fastest way to learn.

## When workflows run (the doorbells)

```yaml
on:
  push:
    branches: [main]        # only when main gets a push
  pull_request:             # when a PR is opened or updated
  schedule:
    - cron: '0 2 * * *'     # 2 AM every day (UTC)
  workflow_dispatch:        # a "Run" button in the Actions tab
```

`push` and `pull_request` cover 90% of real use. `schedule` is for
nightly chores. `workflow_dispatch` is for "run it when I say so."

## Secrets: passwords the robot needs

Workflows often need tokens or passwords (deploy keys, API tokens).
You can't put those in the recipe — the recipe is in git for everyone
to see. **Secrets** solve this:

1. Go to your repo → **Settings → Secrets and variables → Actions** →
   **New repository secret**. Name it `DEPLOY_TOKEN`, paste the value.
2. Use it in the recipe:
   ```yaml
   - run: ./deploy.sh
     env:
       API_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
   ```
3. The robot substitutes the real value at run time. GitHub **masks**
   secrets in logs (shows `***`).

Rules: never `echo` a secret, never commit one to git, and give each
secret the smallest access it needs. (GitHub Enterprise note: your
company may also have **organization** secrets shared across repos —
same idea, wider scope.)

## Runners: GitHub's computers vs yours

**GitHub-hosted** (`runs-on: ubuntu-latest`): GitHub's machines, fresh
every run, free minutes included. Nothing to set up. This is the default
and the right choice most of the time.

**Self-hosted** (`runs-on: [self-hosted, ...]`): *your* machine does the
work. Why you'd want this:
- The job needs your internal network (company servers GitHub can't reach)
- Special hardware (a GPU box, a Mac, a machine with 128GB RAM)
- No minute limits, and the machine already has your tools installed
- Required on GitHub Enterprise when cloud runners can't reach you

Trade-off: you own that machine — its uptime, updates, and security.
Anyone who can run workflows on your repo can run code on your runner,
so only attach self-hosted runners to repos you trust.

## Setting up a self-hosted runner — Linux

On the Linux machine, as a normal (non-root) user:

```bash
# 1. In your repo: Settings → Actions → Runners → "New self-hosted runner"
#    Pick Linux. The page shows you the exact download link and a token.
mkdir actions-runner && cd actions-runner
curl -o runner.tar.gz -L <download-link-from-the-page>
tar xzf runner.tar.gz

# 2. Register (paste the token from the page when asked)
./config.sh --url https://github.com/YOUR-ORG/YOUR-REPO --token <token>

# 3a. Try it: run in the foreground
./run.sh
# 3b. Happy? Install as a service so it survives reboots
sudo ./svc.sh install
sudo ./svc.sh start
```

Then target it with labels:
```yaml
runs-on: [self-hosted, linux]
```

## Setting up a self-hosted runner — Windows

In PowerShell, as Administrator:

```powershell
# 1. In your repo: Settings → Actions → Runners → "New self-hosted runner"
#    Pick Windows. Same deal: download link + token on the page.
mkdir C:\actions-runner; cd C:\actions-runner
Invoke-WebRequest -Uri <download-link-from-the-page> -OutFile runner.zip
Expand-Archive runner.zip .

# 2. Register
.\config.cmd --url https://github.com/YOUR-ORG/YOUR-REPO --token <token>

# 3a. Try it
.\run.cmd
# 3b. As a service (add --runasservice at config time, or re-run config)
.\config.cmd --url https://github.com/YOUR-ORG/YOUR-REPO --token <token> --runasservice --windowslogonaccount "NT AUTHORITY\NETWORK SERVICE"
```

Target it with:
```yaml
runs-on: [self-hosted, windows]
```

## Windows vs Linux: the gotchas

- **Shell:** Linux steps run in `bash`. Windows steps run in
  **PowerShell** by default. Add `shell: bash` to a step to use Git Bash
  on Windows (installs with git for Windows) — this makes simple scripts
  portable.
- **Paths:** `C:\actions-runner\work` vs `/home/user/actions-runner/work`.
  Use forward slashes in YAML; both shells accept them.
- **Line endings:** a script written on Windows may carry `\r\n` and
  confuse Linux bash. If a shell step fails weirdly on Linux only, this
  is suspect number one.
- **Case sensitivity:** `MyFile.txt` and `myfile.txt` are different files
  on Linux, the same file on Windows. Pick lowercase names and stop
  thinking about it.
- **Service accounts:** on Linux the service runs as whoever installed
  it; on Windows pick the service account deliberately — it needs read
  access to the code and write access wherever the job writes.

## One recipe, both systems (matrix)

Test on both OSes without duplicating the recipe:

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - run: python -m pytest
        shell: bash        # same shell on both
```

GitHub runs the job **twice** — once per OS — and shows both results.

## Cookbook: when it breaks

- **Workflow didn't run:** check the `on:` section. Pushed to a branch
  the recipe doesn't listen to? That's the usual cause.
- **Red X:** open the run, click the failed job, read the failed step's
  log from the bottom up. The actual error is usually in the last 20 lines.
- **Runner shows offline:** the service isn't running. Linux:
  `sudo ./svc.sh status` in the runner folder. Windows: check Services
  for "GitHub Actions Runner".
- **Secret not working:** secret names are case-sensitive and must match
  exactly (`${{ secrets.DEPLOY_TOKEN }}` ≠ `deploy_token`).
- **"Permission denied" writing to the repo:** the automatic
  `GITHUB_TOKEN` is read-only by default in some setups. Repo →
  Settings → Actions → General → Workflow permissions → "Read and write".

## GitHub Enterprise in one paragraph

GitHub Enterprise is your company's own private GitHub — same website,
same git, same Actions recipes. Two differences that matter: the runner
registration URL is your company's hostname instead of github.com, and
self-hosted runners are the norm, because GitHub's cloud computers
usually can't reach your internal servers. Everything in this guide
works the same otherwise.

## Golden rules

1. Recipes live in `.github/workflows/`, one concern per file.
2. **Pin your recipe cards:** `actions/checkout@v4`, not `@main`.
   `@main` can change under you overnight.
3. Never print a secret. Ever.
4. Test workflow changes on a branch first — a broken recipe on `main`
   spams red X's on everyone's work.
5. Keep self-hosted runners updated (re-run the download steps every
   few months) and treat them as trusted machines.
6. Read the log from the bottom up. The error is at the end; the
   scrolling above it is just the journey.
