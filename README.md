# Skedda Seat Booking Automation

Automatically book your office seat on [Skedda](https://www.skedda.com) the moment it becomes available.

## How It Works

1. **cron-job.org** sends a trigger to GitHub at your scheduled time (e.g. 00:01 SGT)
2. **GitHub Actions** spins up a server, installs dependencies, and runs the script
3. **The Python script** logs into Skedda, navigates to the target date, and books your preferred seat
4. If your first-choice seat is taken, it automatically tries your next preference

## Features

- Priority-based seat selection (e.g. GEMS 1 > DOM 2 > GEMS 5)
- Books 12 days ahead (configurable) for the earliest possible reservation
- Runs on GitHub Actions — no server or local machine needed
- Credentials stored securely as GitHub Secrets

## Setup Guide

### 1. Fork This Repo

Click the **Fork** button at the top right of this page.

### 2. Find Your Skedda Space IDs

You'll need the space IDs for the seats you want to book:

1. Log into Skedda in your browser
2. Open **Developer Tools** (F12) > **Network** tab
3. Look for a request to `/webs` — the response contains all spaces with their IDs
4. Note down the `id` and `name` for each seat you want

### 3. Set GitHub Secrets

Go to your forked repo > **Settings** > **Secrets and variables** > **Actions** > **New repository secret**

Add the following secrets:

| Secret Name | Example Value | Description |
|---|---|---|
| `SKEDDA_URL` | `https://yourcompany.skedda.com` | Your venue URL (no trailing slash) |
| `SKEDDA_EMAIL` | `you@company.com` | Your Skedda login email |
| `SKEDDA_PASS` | `your_password` | Your Skedda password |
| `SEAT_PRIORITY` | `GEMS 1:1281271,DOM 2:1273200` | Seats in priority order (`name:id`) |

### 4. Configure Your Booking Days

Edit `.github/workflows/book-seat.yml` in your fork to set your preferred days:

```yaml
BOOKING_DAYS: "0,3,4"  # 0=Mon, 1=Tue, 2=Wed, 3=Thu, 4=Fri, 5=Sat, 6=Sun
```

### 5. Set Up cron-job.org (Free Scheduler)

GitHub Actions' built-in cron can delay by 30-60 minutes. For precise timing, use [cron-job.org](https://cron-job.org):

1. Create a free account at **https://cron-job.org**
2. Create a **GitHub Personal Access Token** (Classic) with `repo` scope at **https://github.com/settings/tokens**
3. Create a new cron job for each booking day:

| Field | Value |
|---|---|
| **URL** | `https://api.github.com/repos/YOUR_USERNAME/skedda-seat-booking/actions/workflows/book-seat.yml/dispatches` |
| **Method** | `POST` |
| **Body** | `{"ref":"main"}` |
| **Schedule** | Set to your desired time (e.g. 00:01 SGT) |

4. Add these **headers**:

| Key | Value |
|---|---|
| `Authorization` | `Bearer YOUR_GITHUB_TOKEN` |
| `Accept` | `application/vnd.github.v3+json` |

5. Click **Test** to verify it triggers your workflow

### 6. Verify

- Go to your repo's **Actions** tab to see run history
- Check your Skedda account to confirm the booking was created

## Local Testing

```bash
# Clone and set up
git clone https://github.com/YOUR_USERNAME/skedda-seat-booking.git
cd skedda-seat-booking
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
playwright install chromium

# Copy and fill in your config
cp .env.example .env
# Edit .env with your credentials

# Dry run (simulates without booking)
python book_seat.py --dry-run

# Book a specific date
python book_seat.py --date 2026-04-01

# Book with default settings (12 days ahead)
python book_seat.py
```

## Scheduling Cheat Sheet

To book 12 days ahead, schedule your triggers like this:

| Trigger Day (00:01 SGT) | Target Day (12 days later) |
|---|---|
| Wednesday | Monday |
| Saturday | Thursday |
| Sunday | Friday |

## Troubleshooting

- **Workflow runs but no booking** — Check the Actions log for error details. Common issues: wrong space ID, expired credentials, or target date is not a booking day.
- **404 on cron-job.org test** — Verify your GitHub token has `repo` scope and the URL matches your repo exactly.
- **Login fails** — Skedda may have changed their login page. Check if the email/password field selectors have changed.

## License

MIT
