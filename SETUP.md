# Hosting the Quant Watchlist Report on Your Domain

This bundle is set up to publish the watchlist report to GitHub Pages, served from your custom GoDaddy domain. Once configured:

- Report lives at `https://yourdomain.com` (or `https://watchlist.yourdomain.com`)
- Regenerates automatically every day around 9 PM US Eastern (1:00 UTC)
- Manual refresh available anytime from the GitHub Actions tab
- Free; no servers to maintain; HTTPS handled automatically

## Architecture

```
GitHub Actions          GitHub Pages         GoDaddy DNS
(runs analyzer    →    (serves HTML    →    (your domain
 on schedule)            from /docs)         points here)
```

GitHub Actions runs the Python analyzer on a free Linux VM, commits `report.html` to a `docs/` folder, GitHub Pages auto-publishes that folder, and your domain's DNS records route visitors to GitHub's servers.

## Prerequisites

- A GitHub account (free tier is fine)
- Your GoDaddy domain
- About 15 minutes for setup, plus 10–60 minutes of DNS propagation wait

---

## Setup

### Step 1: Create the GitHub repo

1. On GitHub, click **New repository**.

2. Name it whatever you like (e.g., `quant-watchlist`). **Public** is recommended — it gives you Let's Encrypt SSL automatically. (Private repos require a paid plan for Pages.)

3. Don't initialize with a README, license, or `.gitignore` — this bundle has its own.

4. Push this bundle to the new repo. From the bundle directory:
   ```
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
   git push -u origin main
   ```

   Replace `YOUR-USERNAME` and `YOUR-REPO` with your actual values.

### Step 2: Run the workflow once to populate `docs/`

GitHub Pages can't serve from a folder that doesn't exist yet, so we need to trigger the workflow once before enabling Pages.

1. In the repo, click the **Actions** tab.

2. If prompted "Workflows aren't being run on this repo," click **I understand my workflows, go ahead and enable them**.

3. Click **Generate watchlist report** in the left sidebar.

4. Click **Run workflow** → **Run workflow** (the green button).

5. Wait 2–4 minutes. When the run finishes with a green ✓, your `docs/report.html` exists.

### Step 3: Enable GitHub Pages

1. **Settings** tab of the repo → **Pages** in the left sidebar.

2. Under **Source**, select **Deploy from a branch**.

3. Set:
   - Branch: `main`
   - Folder: `/docs`

4. Click **Save**.

5. Wait ~1 minute, then refresh the page. You should see:
   > Your site is live at `https://YOUR-USERNAME.github.io/YOUR-REPO/`

   Visit that URL to confirm the report renders.

### Step 4: Configure GoDaddy DNS

Decide whether you want the report at the root of your domain or a subdomain. **Subdomain is easier and recommended** if you use the apex domain for anything else (email, another website, etc.).

#### Option A: Subdomain (recommended) — e.g., `watchlist.yourdomain.com`

In GoDaddy:

1. **My Products** → find your domain → **DNS**.

2. Click **Add New Record**:
   - **Type:** `CNAME`
   - **Name:** `watchlist` (or whatever subdomain you want)
   - **Value:** `YOUR-USERNAME.github.io` *(no `https://`, no repo name, just the github.io domain)*
   - **TTL:** `1 Hour`

3. Save.

#### Option B: Apex domain — `yourdomain.com`

GoDaddy GOTCHA: GoDaddy sometimes has a default CNAME for `www` pointing to a parking page (`websitebuilder.com` or similar). **Delete that one first** before adding the GitHub one, or your save will silently fail.

Add **four A records** (all with name `@`, the apex):

| Type | Name | Value             |
|------|------|-------------------|
| A    | `@`  | `185.199.108.153` |
| A    | `@`  | `185.199.109.153` |
| A    | `@`  | `185.199.110.153` |
| A    | `@`  | `185.199.111.153` |

And one CNAME for `www`:

| Type  | Name  | Value                |
|-------|-------|----------------------|
| CNAME | `www` | `YOUR-USERNAME.github.io` |

### Step 5: Tell GitHub about your domain

1. Back in your repo → **Settings → Pages → Custom domain**.

2. Enter your full domain (e.g., `watchlist.yourdomain.com` or `yourdomain.com`). Click **Save**.

3. GitHub creates a `CNAME` file in your repo's root and starts checking DNS. The status flows through:
   - "DNS check in progress" — wait, normal
   - ✓ "DNS check successful" — your DNS resolved
   - "Issuing certificate" — Let's Encrypt is provisioning HTTPS
   - ✓ "HTTPS is supported" — done

   Total: usually 10–30 minutes. Up to 24 hours in worst case for DNS propagation.

4. Once HTTPS is active, **check the "Enforce HTTPS" box** to redirect `http://` to `https://`.

### Step 6: Verify

Visit `https://yourdomain.com` (or `https://watchlist.yourdomain.com`). The bare domain redirects to `report.html` via the small index page the workflow generates. You should see the full quant watchlist report rendered with your domain in the address bar.

---

## Customization

### Change the watchlist

Edit `quant_watchlist/core.py`. Find `DEFAULT_WATCHLIST` (around line 35) — it's a Python list of ticker strings. Modify, save, commit, push:

```
git add quant_watchlist/core.py
git commit -m "Update watchlist"
git push
```

The workflow will trigger automatically because of the `paths:` filter on `quant_watchlist/**`. New report appears in 3–4 minutes.

### Change the schedule

Edit `.github/workflows/build-report.yml`. The cron line:

```yaml
- cron: '0 1 * * *'
```

is in **UTC**. Examples:

| Cron expression       | What it means                                |
|-----------------------|----------------------------------------------|
| `0 1 * * *`           | Daily at 1 AM UTC (~9 PM ET previous day)    |
| `0 22 * * *`          | Daily at 10 PM UTC (~5–6 PM ET, post-close)  |
| `0 1 * * 1-5`         | Weekdays only, 1 AM UTC                      |
| `0 13 * * *`          | Daily at 1 PM UTC (~8–9 AM ET, pre-open)     |
| `0 1,13 * * *`        | Twice daily — 1 AM and 1 PM UTC              |

Use [crontab.guru](https://crontab.guru/) to verify any expression.

### Change the lookback period

In `.github/workflows/build-report.yml`, find the "Generate report" step:

```yaml
- name: Generate report
  run: |
    python -m quant_watchlist --backtest --no-open
```

Add `--period 5y` (or any of `1mo`, `3mo`, `6mo`, `1y`, `2y` (default), `5y`, `10y`, `max`):

```yaml
    python -m quant_watchlist --backtest --period 5y --no-open
```

### Run on demand

GitHub → Actions tab → **Generate watchlist report** → **Run workflow**. Useful when you've updated the watchlist or want a fresh report immediately.

---

## Troubleshooting

### "DNS check unsuccessful" persists more than an hour

Check whether DNS has actually propagated. From any terminal:

```
dig watchlist.yourdomain.com +short
```

If you set up a CNAME for the subdomain, the output should include `YOUR-USERNAME.github.io.` and ultimately resolve to one of `185.199.108.153` through `185.199.111.153`. If you see anything else (like a GoDaddy parking IP), DNS hasn't propagated yet — wait, or check whether the GoDaddy record was actually saved (sometimes the GoDaddy UI looks like it saved but didn't).

### Report shows old data after I changed the watchlist

The workflow only triggers on changes to `quant_watchlist/**`, the workflow file itself, or `requirements.txt`. If you changed something outside those paths, manually trigger via Actions → Run workflow.

### Workflow fails with "yfinance fetch failed"

Yahoo Finance has occasional outages. The workflow will retry on the next schedule. If you want, you can rerun manually from the Actions tab.

### Workflow fails with permission errors on git push

Settings → Actions → General → scroll to **Workflow permissions** → ensure **Read and write permissions** is selected. The first time you set up Actions in a repo, this defaults to read-only.

### "Your site is published at..." but I see a 404

The `docs/` folder may not exist yet. Trigger the workflow manually (Actions tab → Run workflow), wait for it to finish, then refresh.

### The default GitHub Pages URL works but the custom domain doesn't

Most likely the Custom domain field in Settings → Pages is either blank or the DNS hasn't fully propagated. Verify the field shows your domain, click Save again to re-trigger the DNS check, then wait.

---

## What this costs

- **GitHub Pages:** Free for public repos.
- **GitHub Actions:** Free 2,000 minutes/month for private repos; unlimited for public. A daily run uses ~3 minutes, so ~90 minutes/month — well under any limit.
- **Domain:** Whatever you're already paying GoDaddy.
- **Let's Encrypt SSL:** Free and auto-renewed by GitHub.

Total recurring cost above what you already pay: **$0**.

## Privacy note

This is the **watchlist** version of the analyzer — it analyzes a list of well-known publicly traded tickers and contains no personal financial information. The portfolio version (with your actual holdings) intentionally is NOT set up for public hosting. Don't accidentally cross-pollinate them; if you want the portfolio version reachable remotely, that needs authentication (Cloudflare Access on a private repo, or a different hosting approach entirely).
