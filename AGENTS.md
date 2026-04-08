# Career-Ops for Codex

## Data Contract

There are two layers. Read `DATA_CONTRACT.md` for the full list.

**User Layer (NEVER auto-updated, personalization goes HERE):**
- `cv.md`, `config/profile.yml`, `modes/_profile.md`, `article-digest.md`, `portals.yml`
- `data/*`, `reports/*`, `output/*`, `interview-prep/*`

**System Layer (auto-updatable, DO NOT put user data here):**
- `modes/_shared.md`, `modes/oferta.md`, all other modes
- `AGENTS.md`, `*.mjs` scripts, `dashboard/*`, `templates/*`, `batch/*`, `.agents/skills/*`

**THE RULE:** When the user asks to customize anything specific to their search, always write to `modes/_profile.md` or `config/profile.yml`. Never edit `modes/_shared.md` for user-specific content.

## Update Check

On the first message of each session, run the update checker silently:

```bash
bun update-system.mjs check
```

Parse the JSON output:
- `{"status":"update-available",...}` → tell the user:
  > `career-ops update available (v{local} -> v{remote}). Your data (CV, profile, tracker, reports) will NOT be touched. Want me to update?`
  If yes, run `bun update-system.mjs apply`. If no, run `bun update-system.mjs dismiss`.
- `{"status":"up-to-date"}` → say nothing
- `{"status":"dismissed"}` → say nothing
- `{"status":"offline"}` → say nothing

The user can also say "check for updates" or "update career-ops" at any time. To roll back, run `bun update-system.mjs rollback`.

## What Is Career-Ops

AI-powered job search automation built for Codex: pipeline tracking, offer evaluation, CV generation, portal scanning, and batch processing.

### Main Files

| File | Function |
|------|----------|
| `data/applications.md` | Application tracker |
| `data/pipeline.md` | Inbox of pending URLs |
| `data/scan-history.tsv` | Scanner dedup history |
| `portals.yml` | Query and company config |
| `templates/cv-template.html` | HTML template for CVs |
| `generate-pdf.mjs` | Playwright HTML-to-PDF renderer |
| `article-digest.md` | Compact proof points from portfolio |
| `interview-prep/story-bank.md` | Accumulated STAR+R stories |
| `reports/` | Evaluation reports |

### OpenCode Commands

When using OpenCode, slash commands should load the canonical `.agents/skills/career-ops/SKILL.md` router:

| Command | Description |
|---------|-------------|
| `/career-ops` | Show menu or evaluate JD with args |
| `/career-ops-pipeline` | Process pending URLs from inbox |
| `/career-ops-evaluate` | Evaluate job offer (A-F scoring) |
| `/career-ops-compare` | Compare and rank multiple offers |
| `/career-ops-contact` | LinkedIn outreach |
| `/career-ops-deep` | Deep company research |
| `/career-ops-pdf` | Generate ATS-optimized CV |
| `/career-ops-training` | Evaluate course/cert against goals |
| `/career-ops-project` | Evaluate portfolio project idea |
| `/career-ops-tracker` | Application status overview |
| `/career-ops-apply` | Live application assistant |
| `/career-ops-scan` | Scan portals for new offers |
| `/career-ops-batch` | Batch processing |

## First Run - Onboarding

Before doing anything else, silently check for:
1. `cv.md`
2. `config/profile.yml`
3. `modes/_profile.md`
4. `portals.yml`

If `modes/_profile.md` is missing, copy from `modes/_profile.template.md` silently.

If any required file is missing, enter onboarding mode and do not proceed with evaluations or scans until the basics are in place.

### Step 1: CV

If `cv.md` is missing, ask the user whether they want to:
1. paste their CV
2. paste a LinkedIn URL
3. describe their experience and have you draft it

Create clean markdown with standard sections.

### Step 2: Profile

If `config/profile.yml` is missing, copy from `config/profile.example.yml` and gather:
- full name and email
- location and timezone
- target roles
- salary target range

Fill `config/profile.yml`. Put archetypes and targeting narrative in `modes/_profile.md` or `config/profile.yml`, never `modes/_shared.md`.

### Step 3: Portals

If `portals.yml` is missing, copy `templates/portals.example.yml` and offer to customize search keywords for the user's target roles.

### Step 4: Tracker

If `data/applications.md` does not exist, create:

```markdown
# Applications Tracker

| # | Date | Company | Role | Score | Status | PDF | Report | Notes |
|---|------|---------|------|-------|--------|-----|--------|-------|
```

### Step 5: Learn the User

After setup, proactively ask for:
- what makes them unique
- what kind of work excites or drains them
- deal-breakers
- their strongest professional achievement
- articles, projects, or case studies

Store insights in `config/profile.yml`, `modes/_profile.md`, or `article-digest.md`.

### Step 6: Ready

Once setup is complete, confirm they can:
- paste a job URL to evaluate it
- run `/career-ops scan`
- run `/career-ops` to see all commands

Then suggest automation or recurring scans if their client supports it.

## Personalization

This system is designed to be customized in-place by the agent.

Common requests:
- change archetypes → edit `modes/_profile.md` or `config/profile.yml`
- translate modes → edit files in `modes/`
- add companies → edit `portals.yml`
- update profile → edit `config/profile.yml`
- change CV template → edit `templates/cv-template.html`
- adjust scoring weights → use `modes/_profile.md` for personal tuning, or `modes/_shared.md` only when changing shared defaults for everyone

## Language Modes

Default modes are in `modes/` (English). Additional language-specific modes are available:
- `modes/de/` for DACH
- `modes/fr/` for francophone markets
- `modes/pt/` for Brazil

Use the localized modes when the user targets that market or has `language.modes_dir` set in `config/profile.yml`.

## Skill Modes

| If the user... | Mode |
|----------------|------|
| Pastes JD or URL | `auto-pipeline` |
| Asks to evaluate offer | `oferta` |
| Asks to compare offers | `ofertas` |
| Wants LinkedIn outreach | `contacto` |
| Asks for company research | `deep` |
| Preps for interview at specific company | `interview-prep` |
| Wants to generate CV/PDF | `pdf` |
| Evaluates a course/cert | `training` |
| Evaluates portfolio project | `project` |
| Asks about application status | `tracker` |
| Fills out application form | `apply` |
| Searches for new offers | `scan` |
| Processes pending URLs | `pipeline` |
| Batch processes offers | `batch` |

## CV Source of Truth

- `cv.md` in project root is the canonical CV
- `article-digest.md` contains deeper proof points
- never hardcode metrics; read them from those files at evaluation time

## Ethical Use

This system is for quality, not quantity.

- never submit an application without the user reviewing it first
- strongly discourage low-fit applications
- prefer fewer, better applications over mass submission
- respect recruiters' time

## Offer Verification

Never trust generic fetch alone to verify whether an offer is still active.

Preferred path:
1. open the job page in a browser-capable environment
2. inspect the actual job content
3. only footer/navbar without JD means closed; title plus description plus apply means active

Exception for headless batch workers run via `codex exec`: browser tooling may not be available. In that case, use the saved JD text or static fetch fallback and mark verification as unconfirmed in the report.

## Stack and Conventions

- Bun for JS tooling and script execution
- Playwright for PDF generation and job verification
- YAML for config, Markdown for data, HTML/CSS for the CV template
- output in `output/`, reports in `reports/`, JDs in `jds/`, batch artifacts in `batch/`
- report numbering is sequential 3-digit zero-padded
- after each batch of evaluations, run `bun merge-tracker.mjs`
- never create new tracker entries directly in `applications.md` if company + role already exists; update the existing row

### TSV Format for Tracker Additions

Write one TSV file per evaluation to `batch/tracker-additions/{num}-{company-slug}.tsv`. Single line, 9 tab-separated columns:

```
{num}\t{date}\t{company}\t{role}\t{status}\t{score}/5\t{pdf_emoji}\t[{num}](reports/{num}-{slug}-{date}.md)\t{note}
```

Column order:
1. `num`
2. `date`
3. `company`
4. `role`
5. `status`
6. `score`
7. `pdf`
8. `report`
9. `notes`

In `applications.md`, score comes before status. `merge-tracker.mjs` handles the swap.

## Pipeline Integrity

1. never edit `applications.md` to add new entries; write TSV additions and merge them
2. updating status or notes of existing entries is allowed
3. every report must include `**URL:**` in the header
4. all statuses must be canonical per `templates/states.yml`
5. health check: `bun verify-pipeline.mjs`
6. normalize statuses: `bun normalize-statuses.mjs`
7. dedup: `bun dedup-tracker.mjs`

## Canonical States

Source of truth: `templates/states.yml`

| State | When to use |
|-------|-------------|
| `Evaluated` | Report completed, pending decision |
| `Applied` | Application sent |
| `Responded` | Company responded |
| `Interview` | In interview process |
| `Offer` | Offer received |
| `Rejected` | Rejected by company |
| `Discarded` | Discarded by candidate or closed |
| `SKIP` | Not worth applying |
