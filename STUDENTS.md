# Discourse — Student Setup Guide

This is a **course fork of [Discourse](https://github.com/discourse/discourse)** — a forum / community
platform (Ruby on Rails + Ember.js, PostgreSQL, Redis, Sidekiq). Development runs in **Docker** using the
official `discourse/discourse_dev` image (Postgres + Redis + the app all in one container).

📚 **Official documentation:** <https://meta.discourse.org/docs>

> ⚠️ **Windows users — read the Prerequisites first.** You must run with the source on a **Linux-native
> filesystem** (WSL2 or the devcontainer). A plain Windows clone bind-mounted into Docker **does not work**.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| **Docker Desktop** | Running, WSL2 backend on Windows. Give it **≥ 6–8 GB RAM** (Settings → Resources). |
| **Git** | |
| **Disk** | ~15 GB free (the dev image is ~2–3 GB + gems + JS deps). |
| **Windows only:** WSL2 **or** Dev Containers | The source must live on ext4, not `C:\`. Pick one below. |

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

## 5. How to use the tool

Discourse is a full discussion platform: **categories** hold **topics**, topics hold **posts**, with
trust levels, moderation tools, notifications, chat, and a plugin/theme system on top.

- Browse the seeded forum at `http://localhost:3000`; log in as your admin.
- The **admin panel** lives at `http://localhost:3000/admin` — site settings, users, plugins, themes.
- Official documentation (user, admin, and developer guides): <https://meta.discourse.org/docs>.
- Plugin & theme development guides: <https://meta.discourse.org/c/documentation/dev/56>.

---

## 6. How to review the code

Code review is part of the work, not an afterthought:

1. Read the linked issue first, then the diff (GitHub → **Files changed**).
2. Check out the branch locally and run the relevant specs
   (`bin/docker/rspec spec/<area>`).
3. Leave **line comments** for specific problems and finish with a summary review —
   **Approve** or **Request changes**.
4. Look for: correctness, tests covering the new behavior, naming/clarity, and unintended
   changes (lockfiles, generated files, formatting noise).

Every PR needs at least **one teammate approval** before merge — no self-merges. Review the code,
not the person; be specific and constructive.

---

## 7. Pull requests (PRs)

1. Branch off `main`: `git checkout -b feature/<short-name>` (or `fix/<issue-number>-<slug>`).
2. Keep commits small with meaningful messages.
3. Run the relevant specs (section 3) before pushing.
4. Push to the course fork and open the PR against the course fork's `main` — **never upstream**.
5. In the description: what changed, why, how you tested it, and the linked issue (`Closes #12`).
6. Address review comments with follow-up commits (avoid force-pushes during review), then
   re-request review.

---

## 8. Issue resolution process

1. All work is tracked as **GitHub Issues** on the course fork — bug, feature, or task.
2. Before coding: pick or create an issue, get it **assigned** to you, and outline your approach
   in a comment if it's non-trivial.
3. One issue → one branch → one PR, linked with `Closes #<n>` so the issue closes automatically
   on merge.
4. Blocked for more than a day? Say so on the issue (what you tried, where you're stuck) instead
   of going quiet.
5. An issue is **done** when its PR is merged and the behavior is verified.

---

## 9. AI policies

AI assistants (Claude, ChatGPT, Copilot, …) are allowed as a learning and productivity aid,
under these rules:

- **You are the author.** Understand and be able to explain every line you submit — "the AI
  wrote it" is never an explanation.
- **Test before you commit.** Never push AI-generated code you haven't run and tested locally.
- **Disclose it.** Note meaningful AI assistance in the PR description
  (e.g. *"AI-assisted: first draft of the plugin + its specs"*).
- **Protect data.** Never paste secrets, tokens, or private data into AI tools.
