# Glossary

| Term | Meaning |
|------|---------|
| action | A pre-made, reusable chunk of a workflow (e.g. `actions/setup-python@v5`) — someone else's tested recipe card |
| actual state | What is *really* running right now (GitOps compares this to the desired state) |
| artifact | The built thing a pipeline produces: a container image, a package, a zip file |
| branch | A movable label on a commit; `main` is the official one |
| CD (Continuous Deployment/Delivery) | The "applying" half of the pipeline — putting the approved change on real servers |
| CI (Continuous Integration) | The "checking" half of the pipeline — automatically testing every change |
| clone | Download a repo for the first time (`git clone <url>`) |
| commit | One save point — a snapshot of your files plus a message |
| conflict | Git can't auto-combine two edits to the same lines; a human decides |
| declarative | Describing what you *want* ("3 copies running"), not the steps to get there |
| desired state | What git says should be running |
| drift | Reality wandered away from the recipe — someone changed something by hand |
| environment | A separate copy of the system: dev (playground), staging (dress rehearsal), prod (the real thing) |
| event (trigger) | The thing that starts a workflow: a push, a pull request, a schedule, a manual button |
| fetch | Ask GitHub "what's new?" without downloading files yet |
| HEAD | "Where I am right now" — the current commit |
| issue | A to-do item on a repo: bug report, feature request, question |
| job | One chunk of work in a workflow; each job runs on its own runner |
| merge | Fold one branch's commits into another |
| pipeline | The robot assembly line — automatic steps that check and then apply a change |
| promotion | Moving a change up the ladder: dev → staging → prod |
| pull request (PR) | A proposal to fold one branch's commits into another, with review and automated checks |
| reconciliation loop | The GitOps robot's endless chore: compare desired state to actual state, fix what doesn't match |
| release | A packaged, labeled version of a repo — often where installers and changelogs live |
| remote | The shared copy of the repo (GitHub). Nicknamed `origin` by default |
| repository (repo) | The project folder plus its full history of commits |
| rollback | Undo — go back to the earlier commit that worked |
| runner | The computer that runs a workflow's jobs — GitHub's, or your own (self-hosted) |
| scope | What a token is allowed to do (read repos, write PRs, …) |
| secret | A password or token a workflow needs, stored in GitHub — never committed to the repo |
| stash | Temporarily shelve unfinished changes |
| step | One instruction inside a job; steps run in order, top to bottom |
| tag | A labeled snapshot of the repo, e.g. `v1.2.3` |
| token | A long secret string that proves it's you, used instead of your password |
| workflow | The recipe — a YAML file in `.github/workflows/` that says "when X happens, do Y" |
| workflow run | One execution of a workflow |
