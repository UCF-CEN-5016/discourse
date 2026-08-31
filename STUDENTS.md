# Discourse — Student Setup Guide

This is a **course fork of [Discourse](https://github.com/discourse/discourse)** — a forum / community
platform (Ruby on Rails + Ember.js, PostgreSQL, Redis, Sidekiq). Development runs in **Docker** using the
official `discourse/discourse_dev` image (Postgres + Redis + the app all in one container).

#demo

📚 **Official documentation:** [https://meta.discourse.org/docs](https://meta.discourse.org/docs)

> ⚠️ **Windows users — read the Prerequisites first.** You must run with the source on a **Linux-native
> filesystem** (WSL2 or the devcontainer). A plain Windows clone bind-mounted into Docker **does not work**.

---

## 1. Prerequisites

| Requirement                                              | Notes                                                                                      |
| -------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| **Docker Desktop**                                 | Running, WSL2 backend on Windows. Give it**≥ 6–8 GB RAM** (Settings → Resources). |
| **Git**                                            |                                                                                            |
| **Disk**                                           | ~15 GB free (the dev image is ~2–3 GB + gems + JS deps).                                  |
| **Windows only:** WSL2 **or** Dev Containers | The source must live on ext4, not`C:\`. Pick one below.                                  |

> 💡 **New to Docker?** Discourse's dev environment runs entirely inside a container. The [Get Started guide](https://docs.docker.com/get-started/) is a good orientation — you mostly just need `docker start`/`docker stop` day-to-day.

> 💡 **New to Ruby on Rails?** Discourse's backend is a Rails app. Skim [Getting Started](https://guides.rubyonrails.org/getting_started.html) and [Active Record Basics](https://guides.rubyonrails.org/active_record_basics.html) in the Rails Guides before touching the backend.

> 📖 **Developer guides:** [meta.discourse.org — developer documentation](https://meta.discourse.org/c/documentation/dev/56) is the authoritative source for architecture notes, plugin development, and core internals. Start there before picking a task.

**Windows — pick ONE native-filesystem option:**

- **WSL2 (recommended):** `wsl --install -d Ubuntu`, open the Ubuntu terminal, and clone into your Linux
  home (`~`), **NOT** `/mnt/c/...`. Enable Docker Desktop → Settings → Resources → **WSL Integration** for
  Ubuntu so `docker` works inside it.
- **Dev Containers:** open the repo in VS Code with the **Dev Containers** extension → "Reopen in Container"
  (uses the repo's `.devcontainer/`).

**macOS / Linux:** no extra step — clone anywhere and continue.

---

## 2. Get it running

Run these in a normal terminal (Windows: the **Ubuntu/WSL2** terminal, in `~`):

```bash
git clone https://github.com/musta55/discourse.git
cd discourse

# One-shot setup: pulls the dev image, installs gems + JS deps, creates & migrates the
# dev/test databases, and creates your admin user (it will PROMPT for email + password).
bin/docker/boot_dev --init

# Seed sample categories, topics, tags, and users:
bin/docker/rake dev:populate

# Start the dev server (leave it running; Ctrl-C to stop):
bin/docker/shell            # opens a shell inside the container
#   …now inside the container:
bin/dev
```

Then open **http://localhost:3000** and log in with the admin email/password you set during `--init`.
(First boot compiles the frontend — give it a couple of minutes. You should see
`Pitchfork ready on http://localhost:3000`.)

Everyday container control (from the host):

```bash
docker start discourse_dev     # resume after a reboot
docker stop  discourse_dev     # stop (data is preserved)
```

---

## 3. Run the tests

```bash
# Backend (RSpec) — run targeted specs; the full suite is very large:
bin/docker/rspec spec/models/category_spec.rb        # one file
bin/docker/rspec spec/models                          # a folder
bin/docker/rspec spec/requests/...                    # request specs

# Frontend (Ember QUnit):
bin/docker/rake qunit:test
```

> **Known pre-existing failure (don't chase it):** on this baseline, `spec/models/post_spec.rb`
> has **one** failing example — `Post#cannot_permanently_delete_reason` (`RateLimiter.time_left`).
> It's a bug that predates your work, reproducible in isolation. Everything else in the core model
> specs passes (~1651/1652).

---

## 4. Inspect / check the seeded data

```bash
# Rails console:
bin/docker/rails c
#   > User.count
#   > Category.pluck(:name)
#   > Topic.order(created_at: :desc).limit(5).pluck(:title)

# PostgreSQL shell:
bin/docker/psql
#   => SELECT count(*) FROM users;   SELECT name FROM categories LIMIT 10;

# Or via the running site's JSON API:
#   http://localhost:3000/site.json
#   http://localhost:3000/categories.json
#   http://localhost:3000/latest.json

# Re-seed at any time:
bin/docker/rake dev:populate
```

After `dev:populate` you'll have ~30 categories, ~30 users, and topics with posts to click around.

---

---

## Contributing workflow

All team members have write access to this repository, so the team uses a **branch-based** workflow — not forks. Here is the background and the commands.

**Why not forks?** Forking is the standard model for contributing to open-source projects where you _don't_ have write access: you fork to your own GitHub account, clone your fork, and open a PR from your fork back to the original. You will encounter this when contributing to the upstream project. But for your course team — where everyone has write access to the shared repo — it just adds confusion: two clones on your machine, two remotes to keep in sync, merge conflicts that are harder to reason about.

**Branch-based workflow** is what most professional teams use internally. You clone the shared repo once, create a short-lived branch for each issue, push the branch back to the same repo, and open a PR from that branch into `main`. One clone, one remote, full PR workflow.

### For each issue you work on

```bash
# One-time setup: clone the team repo (skip if already done)
git clone https://github.com/CSCI-435-SE/discourse.git
cd discourse

# Before starting each issue: make sure you are on a fresh main
git checkout main
git pull origin main

# Create a branch named for the issue
git checkout -b feat/issue-17-dark-mode      # new feature
git checkout -b fix/issue-42-toast-dismiss   # bug fix

# ... make your changes, run tests ...

# Stage and commit
git add <the files you changed>
git commit -m "feat: add dark mode toggle (#17)"

# Push the branch to the team repo
git push origin feat/issue-17-dark-mode
```

After pushing, GitHub shows a **"Compare & pull request"** banner on the repository page. Click it to open a PR from your branch into `main`. Fill in the description (what changed and why), reference the issue (`Closes #17`), and request a review from a teammate.

**Branch naming:**

| Prefix | Use for |
|---|---|
| `feat/issue-<N>-short-description` | new features |
| `fix/issue-<N>-short-description` | bug fixes |
| `chore/short-description` | docs, config, dependency updates |

> ⚠️ **`main` is protected — direct pushes are blocked.** All changes go through a reviewed PR. If you accidentally commit to `main` locally, move your changes to a branch before pushing:
>
> ```bash
> git checkout -b fix/issue-42-my-fix   # create branch from your current state
> git checkout main
> git reset --hard origin/main          # revert local main to match remote
> ```

**After your PR is merged**, delete the branch to keep the repo tidy:

```bash
git checkout main
git pull origin main
git branch -d feat/issue-17-dark-mode
```

## 5. Project documentation & policies (required reading)

📚 **Official documentation:** [https://meta.discourse.org/docs](https://meta.discourse.org/docs) — user, admin, and developer
guides; plugin & theme development at
[meta.discourse.org/c/documentation/dev/56](https://meta.discourse.org/c/documentation/dev/56).

Discourse has its own established contribution processes. They are **not restated here** — you are
responsible for finding, reading, and following them from the sources below:

| You must take care of                  | Where to find it                                                                                                                                                                                                               |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| How to use the tool                    | [https://meta.discourse.org/docs](https://meta.discourse.org/docs)                                                                                                                                                              |
| Code review process                    | [Discourse Development Contribution Guidelines](https://meta.discourse.org/t/discourse-development-contribution-guidelines/3823)                                                                                                |
| Bug / issue resolution process         | [How to make bug reports for Discourse](https://meta.discourse.org/t/how-to-make-bug-reports-for-discourse/33070) · [How to request new features](https://meta.discourse.org/t/how-to-request-new-features-for-discourse/32986) |
| Pull request conventions & PR policies | [CONTRIBUTING.md](CONTRIBUTING.md) (incl. the **CLA** requirement) + the [contribution guidelines](https://meta.discourse.org/t/discourse-development-contribution-guidelines/3823)                                        |
| AI policies                            | [AI-AGENTS.md](AI-AGENTS.md) (this repo's AI-agent guide)                                                                                                                                                                       |
