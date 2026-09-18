**Currently in beta testing only, and repo not available publicly.**

# Bottle-Logger-Public
Custom mobile web page used to scan day care documents and render the bottle logs into a baby tracking app.

<img width="150" height="326" alt="image0" src="https://github.com/user-attachments/assets/be8c8d29-6ef1-4022-84a5-4abdae625510" />

<img width="150" height="326" alt="image1" src="https://github.com/user-attachments/assets/43141ed7-c8e1-4db3-8798-7cd9da4c0882" />

<img width="150" height="326" alt="image2" src="https://github.com/user-attachments/assets/c911765b-c181-4aa5-9c84-e051447e2cea" />



# Bottle Log

Take a photo of the daycare's daily bottle schedule sheet, review what it parsed, and log each bottle straight into [Huckleberry](https://huckleberrycare.com/) — no manual typing.

## How it works

1. Open the app (a private, secret-URL page — no login) on your phone
2. Take a photo of the sheet, or select one from your library
3. Claude's vision API reads the sheet and extracts just the bottle rows (time + ounces), ignoring everything else on it
4. Review each parsed entry — confirm, edit (with a native time picker and +/- oz stepper), or delete any row
5. Hit **Confirm All & Log to Huckleberry** — each entry gets written to Huckleberry via its (unofficial) Firestore API

## Project structure

| File | Purpose |
|---|---|
| `web_app.py` | Flask app — the secret-URL page, review UI, and confirm/log flow |
| `extract_bottles.py` | Vision extraction: photo → structured JSON of bottle entries |
| `pipeline.py` | Shared logic: resolving parsed times into real datetimes, writing to Huckleberry |
| `requirements.txt` | Python dependencies |

## Setup

### Requirements
- Python 3.14+
- An [Anthropic API key](https://console.anthropic.com/)
- A Huckleberry account (email/password)

### Environment variables

| Variable | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Your Anthropic Console API key |
| `HUCKLEBERRY_EMAIL` | Huckleberry account email |
| `HUCKLEBERRY_PASSWORD` | Huckleberry account password |
| `HUCKLEBERRY_TZ` | Timezone for logged entries, e.g. `America/New_York` |
| `HUCKLEBERRY_BOTTLE_TYPE` | `Formula` or `Breast Milk` |
| `APP_SECRET_PATH` | A long random string that gates access to the app (acts as the "auth") — generate one with `python -c "import secrets; print(secrets.token_urlsafe(24))"` |

### Run locally

```bash
pip install -r requirements.txt
python web_app.py
```

Then visit `http://localhost:5000/<APP_SECRET_PATH>/`. To access it from your phone during local testing, expose it with [ngrok](https://ngrok.com/): `ngrok http 5000`.

### Deploy (Render)

1. Push this repo to GitHub
2. Create a new Web Service on [Render](https://render.com/), connected to the repo
3. Build command: `pip install -r requirements.txt`
4. Start command: `gunicorn web_app:app`
5. Add the environment variables above in Render's dashboard
6. Visit `https://<your-render-url>/<APP_SECRET_PATH>/`

A `/ping` route is included (unauthenticated, returns `"ok"`) for use with an external scheduler like [cron-job.org](https://cron-job.org/) to keep a free-tier Render instance awake during a specific daily window, since free instances sleep after 15 minutes of inactivity.

## Security note

Access is gated by an unguessable secret URL path rather than a login — reasonable for a low-stakes personal tool like this, but it's obscurity, not real authentication. Don't reuse this pattern for anything sensitive. Never commit real values for any of the environment variables above to the repo.
