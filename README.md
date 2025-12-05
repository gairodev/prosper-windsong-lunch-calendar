## Windsong Elementary Lunch Calendar (Nutrislice → iCal)

This repository generates an iCalendar (.ics) file with Windsong Elementary lunch menus by fetching data from the Nutrislice API and converting each school day’s menu into an all‑day calendar event.

The output file `windsong_lunch.ics` is committed back to the repo on a schedule, so anyone can subscribe to the calendar via a stable URL.

### What this repo does

- Fetches lunch menus from Nutrislice for a rolling one‑year window starting today.
- Creates one all‑day event per date. The event description lists the menu items.
- Writes the result to `windsong_lunch.ics` at the repo root.
- A GitHub Actions workflow runs weekly and on demand to refresh the file and commit updates.

## Quick start (local)

1) Use Python 3.12+ (matches CI). Create a virtual environment and install deps:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

2) Generate the calendar:

```bash
python generate_calendar.py
```

This creates `windsong_lunch.ics` in the repo root. Import it into your calendar app, or host it somewhere and subscribe by URL.

### Subscribe by URL

If this repo is hosted on GitHub, you can subscribe directly to the “raw” file URL. Replace placeholders with your org/user and repo:

- Markdown link format: `[Raw windsong_lunch.ics](https://raw.githubusercontent.com/<org-or-user>/<repo>/main/windsong_lunch.ics)`
- Example template: `https://raw.githubusercontent.com/<org-or-user>/<repo>/main/windsong_lunch.ics`

Most calendar apps support adding a calendar by URL. Paste the raw link; updates are pulled automatically when the workflow commits new content.

## How it works

- Script: `generate_calendar.py`
  - Builds the Nutrislice weeks endpoint from a district subdomain, school slug, and meal type.
  - Fetches week JSON, collects day menus for dates within the next year.
  - Produces a standards‑compliant `.ics` with one all‑day event per date:
    - `SUMMARY`: “Windsong Lunch – Mon DD”
    - `DESCRIPTION`: bullet‑style list of menu items (wrapped to safe line lengths)
    - If no items are listed by Nutrislice, the event notes that explicitly.

- Dependencies: `requests`
  - See `requirements.txt`. Install with `pip install -r requirements.txt`.

- Output: `windsong_lunch.ics`
  - Overwritten each run with fresh data.

## Scheduled updates (GitHub Actions)

Workflow: `.github/workflows/update-calendar.yml`

- Runs every Monday at 09:00 UTC and can also be triggered manually.
- Steps:
  - Checkout
  - Set up Python 3.12
  - `pip install -r requirements.txt`
  - `python generate_calendar.py`
  - Copies the generated file into `docs/windsong_lunch.ics`
  - Commits updates if either file changed

This keeps the calendar current without manual intervention.

## Configuration

Adjust these constants in `generate_calendar.py` if needed:

- `DISTRICT_SUBDOMAIN`: Nutrislice district subdomain (e.g., `"prosperisd"`).
- `SCHOOL_SLUG`: School slug (e.g., `"windsong-elementary"`).
- `MEAL_TYPE`: Menu type path segment (commonly `"lunch"`, but districts may vary).
- `CALENDAR_NAME`: Displayed calendar name in the `.ics`.
- `PRODID`: iCal producer identifier string.

If your district’s Nutrislice deployment uses a different endpoint pattern or meal slug, tweak `build_weeks_url` and/or `MEAL_TYPE`. If the request returns 404, open your browser’s dev tools on the Nutrislice site and copy the menu API URL pattern it uses.

## Notes and limitations

- Time zone: Events are all‑day, so no time zone handling is required by consumers.
- Coverage: The script targets a rolling 365‑day window from today.
- Gaps: If Nutrislice returns no items for a date, the event clarifies “No Menu Listed”.
- API reliability: Network hiccups are logged with a warning; missing weeks are skipped.
- Formatting: Lines are wrapped in the iCal description to remain RFC‑friendly.

## Project structure

- `generate_calendar.py` — main script (fetch, parse, build `.ics`).
- `requirements.txt` — Python dependencies (minimal: `requests`).
- `.github/workflows/update-calendar.yml` — weekly auto‑update workflow.
- `windsong_lunch.ics` — generated calendar file (committed by CI).
 - `docs/index.md` — GitHub Pages site with subscribe instructions and a live link.
 - `docs/windsong_lunch.ics` — copy of the calendar served by GitHub Pages.

## Contributing

PRs welcome for:

- Additional meal types or multiple calendars.
- Alternative districts/schools via config.
- Better error handling or richer event metadata.

Open an issue if Nutrislice changes break the endpoint format; include the district and a sample working API URL from your browser network tab.

## GitHub Pages

To host a small site with subscribe instructions and a stable link:

- In repository Settings → Pages, set “Source” to “Deploy from a branch”.
- Choose Branch: `main`, Folder: `/docs`.
- Save. Your site will be available at a URL like:
  - `https://<org-or-user>.github.io/<repo>/`
  - The calendar file will be at: `https://<org-or-user>.github.io/<repo>/windsong_lunch.ics`

The workflow keeps `docs/windsong_lunch.ics` current. The `docs/index.md` page includes the link and copy‑paste steps for Google Calendar, Apple Calendar, and Outlook.
