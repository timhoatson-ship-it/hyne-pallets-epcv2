# Security Setup Guide

This guide walks you through securing the Hyne Pallets app. No coding required — just Railway dashboard and a few Git commands.

---

## 1. Set JWT_SECRET in Railway

The app **will not start** without this variable.

1. Open your Railway project dashboard
2. Click on your service → **Variables** tab
3. Click **New Variable**
4. Set:
   - **Name**: `JWT_SECRET`
   - **Value**: A long random string (at least 32 characters)
5. To generate a strong value, run this in any terminal:
   ```
   python3 -c "import secrets; print(secrets.token_hex(32))"
   ```
   Copy the output and paste it as the value.
6. Click **Save** — Railway will redeploy automatically.

> If `JWT_SECRET` was previously set to the old default (`hyne_pallets_secret_2026_CHANGE_ME`),
> **change it immediately**. Anyone who saw the README knows that value.

---

## 2. Set SMTP_PASSWORD (if using email notifications)

The SMTP mail server password is now stored as an environment variable, not in the database.

1. In Railway → **Variables** tab
2. Add `SMTP_PASSWORD` with your mail server password
3. The host/port/username are still configured in Admin → Email Config

> Previously the SMTP password was stored in the database with XOR obfuscation.
> That has been removed — the env var is the only source now.

---

## 3. Set Default Passwords

These are only used when the database is created for the first time (i.e., the `users` table is empty).

| Variable | What it sets |
|----------|-------------|
| `ADMIN_DEFAULT_PW` | Password for the admin account (tim@hynepallets.com.au) |
| `DEFAULT_USER_PW` | Password for all other seed accounts (sarah, mike, dave, etc.) |

Set these in Railway Variables **before first deployment** (or before deleting the database to reset).

If you don't set them, the app falls back to weak built-in defaults — change passwords via Admin panel immediately after first login.

---

## 4. Remove data.db from Git Tracking

The database file `data.db` is already listed in `.gitignore`, but if it was committed before that rule existed, Git still tracks it. To remove it:

```bash
cd your-repo-folder
git rm --cached data.db
git commit -m "Remove data.db from source control"
git push
```

This removes the file from Git **without deleting your local copy**. The `.gitignore` rule will prevent it from being re-added.

> **Important**: Even after removal, the old file remains in Git history.
> See Section 7 below on how to fully purge it if needed.

---

## 5. What Needs Rotating Now

If any of these were previously visible in the repo (README, committed code, or database), treat them as compromised:

| Secret | Where to change it |
|--------|--------------------|
| `JWT_SECRET` | Railway Variables tab → set a new random value |
| Admin password | Log in → Admin panel → change password |
| All user passwords | Admin panel → reset each user's password |
| Floor PINs (123456, 234567, etc.) | Admin panel → change each floor worker's PIN |
| Driver PINs (111111, 222222, etc.) | Admin panel → change each driver's PIN |

After rotating `JWT_SECRET`, all existing login sessions become invalid — users will need to log in again. This is expected and correct.

---

## 6. Ongoing Rules

- **Never commit** `.env` files, database files, or secrets to the repo.
- **Never put** passwords, PINs, or secret keys in the README or code comments.
- Set all secrets via **Railway Variables** (or `.env` file locally, which is gitignored).
- When adding new team members, create accounts with strong passwords via Admin, not via seed data.

---

## 7. Purging Secrets from Git History (Optional, Advanced)

Removing a file with `git rm --cached` stops tracking it going forward, but old commits still contain the file. If `data.db` or old README versions with passwords are in the history, they can still be viewed.

To fully remove them, you would use `git filter-repo` (preferred) or `BFG Repo-Cleaner`:

```bash
# Install git-filter-repo (once)
pip install git-filter-repo

# Remove data.db from all history
git filter-repo --path data.db --invert-paths

# Force-push the rewritten history
git push --force
```

**Warning**: This rewrites all commit hashes. All collaborators must re-clone the repo after this.

Only do this when you're ready and have confirmed no one else has pending work on the repo.

---

## Questions?

If anything is unclear, ask Tim or the developer who set up the system.
