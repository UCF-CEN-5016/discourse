# Discourse — Student Setup Guide

This is a **course fork of [Discourse](https://github.com/discourse/discourse)** — a forum / community
platform (Ruby on Rails + Ember.js, PostgreSQL, Redis, Sidekiq). Development runs in **Docker** using the
official `discourse/discourse_dev` image (Postgres + Redis + the app all in one container).

> ⚠️ **Windows users — read the Prerequisites first.** You must run with the source on a **Linux-native
> filesystem** (WSL2 or the devcontainer). A plain Windows clone bind-mounted into Docker **does not work**
> (see "Why not a plain Windows clone").

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

## 5. Doing your work

```bash
git checkout -b feature/<your-feature>
# …make changes (hot-reload picks up most edits)…
bin/docker/rspec spec/...          # test your change
git commit -am "feat: <what you did>"
git push origin feature/<your-feature>     # open a PR against the course fork
```

**Good starter areas** (this is a large, mature codebase — stay near the edges):
- **Plugins** (`plugins/`) — small, self-contained features. This is the recommended place to work.
- **Theme / customization** tasks.
- Small **moderation / notification** improvements.

Avoid rewriting core forum logic.

---

## Why not a plain Windows clone (`C:\…`)?

If you clone on Windows and bind-mount `C:\…` into Docker, two things break:
1. **Line endings:** Git on Windows checks out `bin/*` scripts with CRLF, so their `#!/usr/bin/env ruby`
   shebang fails in the Linux container (`/usr/bin/env: 'ruby\r': No such file or directory`).
2. **The dev web server won't boot:** its file-watchers use inotify, which doesn't work over the
   Windows→Docker filesystem mount — pitchfork's workers crash on startup.

Both disappear when the source is on a Linux-native filesystem (WSL2 or the devcontainer), which is why
those are required on Windows. If you ever see the `'ruby\r'` error, you're on a `C:\` clone — switch to WSL2.
