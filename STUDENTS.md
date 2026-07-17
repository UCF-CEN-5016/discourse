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

## 5. Project documentation & policies (required reading)

📚 **Official documentation:** <https://meta.discourse.org/docs> — user, admin, and developer
guides; plugin & theme development at
[meta.discourse.org/c/documentation/dev/56](https://meta.discourse.org/c/documentation/dev/56).

Discourse has its own established contribution processes. They are **not restated here** — you are
responsible for finding, reading, and following them from the sources below:

| You must take care of | Where to find it |
| --- | --- |
| How to use the tool | <https://meta.discourse.org/docs> |
| Code review process | [Discourse Development Contribution Guidelines](https://meta.discourse.org/t/discourse-development-contribution-guidelines/3823) |
| Bug / issue resolution process | [How to make bug reports for Discourse](https://meta.discourse.org/t/how-to-make-bug-reports-for-discourse/33070) · [How to request new features](https://meta.discourse.org/t/how-to-request-new-features-for-discourse/32986) |
| Pull request conventions & PR policies | [CONTRIBUTING.md](CONTRIBUTING.md) (incl. the **CLA** requirement) + the [contribution guidelines](https://meta.discourse.org/t/discourse-development-contribution-guidelines/3823) |
| AI policies | [AI-AGENTS.md](AI-AGENTS.md) (this repo's AI-agent guide) |
