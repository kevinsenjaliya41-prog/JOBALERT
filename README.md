# 🚗 Job Alert Bot — Get Yours Running in ~30 Minutes

Your personal job-hunting robot: every 2 hours it scans LinkedIn, job boards,
and company career pages (Bosch, BMW, Mercedes, Wayve…), grades every posting
0–100 against **your actual CV** using a free AI, and sends the good ones to
your Telegram. Completely free to run. No laptop needed after setup.

- 🟢 score ≥75 → strong match, alerted immediately
- 🟡 score ≥50 → worth a look, alerted
- 🔴 below → archived quietly, one summary per evening
- 😴 silent 23:00–06:00 German time

---

## Setup — do these steps in order

### 0. Get this code as YOUR repo (2 min)
1. Click **Use this template → Create a new repository** (top right of this page).
2. Name it whatever you like, and set visibility to **Private** — your CV will
   live inside it.

### 1. Make it yours (10 min — the most important step)
Edit **ONE file**: `cv.md` (click it on GitHub → pencil icon → commit).
Paste your real CV into the structure and fill the **Additional
Information** section at the bottom (what you're looking for, interests,
locations, things to avoid).

That's it. On your first real run, the system reads `cv.md` and
**auto-generates your entire matching profile** — your field, skills,
target job titles, keyword filters, and job-board search queries — and
commits it back to your repo as `profile.yaml`. This works for ANY field:
engineering, medicine, law, CS, finance, math…

Afterwards you can fine-tune the generated `profile.yaml` by hand any time
(it won't be overwritten), or regenerate it after a CV update by running
the workflow… or locally: `python main.py --personalize`.

Optional power-tuning in `config.yaml`:
- `platforms.companies.targets` — track your field's employers' career
  portals directly (any company on SmartRecruiters, Greenhouse, or Lever)

### 2. Groq API key — the free AI grader (3 min)
1. Sign up at https://console.groq.com (Google login works).
2. **API Keys → Create API Key** → copy the `gsk_...` value (shown only once).

### 3. Telegram bot — your alerts (5 min)
1. In Telegram, message **@BotFather** → `/newbot` → pick a name and a
   username ending in `bot` → copy the **bot token** (`123456:AAxx...`).
2. **Open a chat with your new bot and send it any message** (mandatory —
   bots can't message you first).
3. In a browser, open
   `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates`
   (with your token in place of `<YOUR_TOKEN>`) and find
   `"chat":{"id":123456789` — that number is your **chat ID**.

### 4. Add the secrets to your repo (3 min)
On your repo page: **Settings → Secrets and variables → Actions →
New repository secret**. Add these, names exactly as written:

| Name | Value |
|---|---|
| `GROQ_API_KEY` | your `gsk_...` key |
| `TELEGRAM_BOT_TOKEN` | your bot token |
| `TELEGRAM_CHAT_ID` | your chat ID number |

Optional extras: `ADZUNA_APP_ID` + `ADZUNA_APP_KEY` (free at
https://developer.adzuna.com/signup — adds a solid German job board) and
`LINKEDIN_LI_AT` (your LinkedIn `li_at` cookie for richer results).

⚠️ Never put keys in any file. The system refuses to start if `config.yaml`
looks like it contains one, and refuses real runs without the Telegram secrets.

### 5. Launch (2 min)
1. **Actions** tab → enable workflows if asked.
2. Click **job-alert** (left sidebar) → **Run workflow** dropdown → tick
   *"Run even during quiet hours"* → **Run workflow**.
3. Watch the run go green (~2–6 min). Your first digest arrives on Telegram —
   it's long (the whole current backlog); after this only new postings alert.

Done. The schedule takes over: every 2 hours, 06:00–23:00 German time.

---

## Daily life with the bot

- **Silence = nothing new.** The bot only speaks when it found something.
- **Every evening ~21:00** you get a one-line recap of what it rejected and why.
- **`state/archive/<date>/`** in your repo holds everything it saw each day.
- Small `chore: update state [skip ci]` commits appear after each run — that's
  the bot remembering which jobs it already showed you. Normal.
- Tune alert volume with `tier_thresholds` in `profile.yaml`
  (raise `strong`/`worth_look` = fewer, better alerts).

## Troubleshooting

| Symptom | Cause / fix |
|---|---|
| Run red ❌, "TELEGRAM_... not set" | A secret is missing or misnamed — step 4 |
| Run green but no Telegram | Genuinely nothing new — check `state/archive/` |
| "Groq daily token quota reached" in log | Free AI quota used up; scores fall back to rules until it refills (~24h) — harmless |
| 9-second runs at night | Quiet hours working as designed |
| LinkedIn shows ⚠ in the digest footer | GitHub's servers got rate-limited by LinkedIn this run — other sources still cover you |

## How it works

```
fetch (6 sources, isolated) → dedup (30-day history)
  → rule pre-filter (free)  → domain classify (rules first, AI for ambiguous)
  → Groq scores 0-100 against YOUR cv.md (≤60 calls/run, paced for free tier)
  → tier → Telegram → archive → commit state
```

Built by Kevin & Meet for automotive engineering students in Germany —
adaptable to any field by editing `profile.yaml`, `cv.md`, and the queries in
`config.yaml`.
