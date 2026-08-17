# Huggingface-Talent-Radar-with-Claude-n8n-Airtable

Automated Hugging Face talent sourcing with n8n + Claude + Airtable.

![Hugging Face Talent Radar Workflow](HugginFace%20n8n%20Talent%20Radar.png)

## Why Hugging Face

Hugging Face is where AI engineers and researchers publish the actual models they build: not papers about them and not social posts about them, but working model artifacts that developers use in production.

Download counts are a strong proxy for real-world impact. If a model has hundreds of thousands of downloads, it suggests broad adoption and practical relevance beyond a CV bullet point.

This repository gives you a weekly n8n workflow that searches Hugging Face using four methods, cross-references authors on GitHub, evaluates profiles with Claude, and stores qualified candidates in Airtable with personalized outreach hooks informed by model impact signals.

## Who this is for (and not for)

**This is for you if:**
- You recruit ML/AI engineers and want a repeatable Hugging Face sourcing system.
- You want AI-assisted candidate scoring and personalized outreach hooks.
- You're comfortable connecting APIs in n8n (Hugging Face, Claude, GitHub, Airtable).

**This is NOT for you if:**
- You only hire non-technical roles.
- You cannot use external APIs due to internal compliance/policy constraints.

## The 4 search engines this workflow runs

The workflow runs all 4 searches every week and combines the results into one ranked talent pipeline.

### Search 1 — By Task
Finds the most-downloaded models for a specific task (e.g. `text-generation`). Download count is a strong signal of real-world impact, so the most-adopted work surfaces first.

### Search 2 — Trending
Finds models gaining the most likes in the last 7 days — engineers whose work is getting attention right now, not just all-time leaders.

### Search 3 — By Org
Pulls the most-downloaded models published under a specific organisation — competitor and peer-company mining. Set your own target org in the search URL.

### Search 4 — By Library
Finds the most-downloaded models built with a specific library (e.g. `transformers`) for a given task — engineers working in a specific technical stack you care about.

Note: See how customize the searches for your role below. 

## How the workflow operates

1. A weekly schedule trigger starts the workflow (Mondays at 07:00).
2. The four Hugging Face searches run in parallel and are merged.
3. A code node extracts and deduplicates authors, then caps the candidate list.
4. Each author is processed one by one; their HF profile is fetched (org accounts 404 and are skipped).
5. GitHub is searched for the author's profile as a cross-reference.
6. A code node formats the combined data and enforces a per-run test budget.
7. Claude scores each candidate against your target role and returns structured JSON.
8. A parse step extracts the JSON; an IF node keeps only candidates above the score threshold.
9. Qualified candidates are saved to Airtable.

## 5 accounts you need to create

1. **n8n** (n8n.io) — the automation engine that connects everything
2. **Hugging Face** (huggingface.co) — your source of candidates
3. **Anthropic** (console.anthropic.com) — Claude AI scoring
4. **GitHub** (github.com) — author profile cross-reference
5. **Airtable** (airtable.com) — your candidate database

## How to get each API key

- **Hugging Face**: Log in → Settings → **Access Tokens** → *New token* (read scope is enough). In n8n, create a **Hugging Face** credential and paste the token.
- **Anthropic (Claude)**: console.anthropic.com → **API Keys** → *Create Key*. In n8n, create an **Anthropic** credential with the key. Make sure your account has billing/credits enabled.
- **GitHub**: GitHub → Settings → Developer settings → **Personal access tokens** → *Fine-grained token* (public read access is enough for user search). In n8n, create a **GitHub** credential with the token.
- **Airtable**: airtable.com/create/tokens → *Create token* with `data.records:write` and `schema.bases:read` scopes, granted to your base. In n8n, create an **Airtable Personal Access Token** credential.

> Never paste API keys into the workflow JSON or into `PROMPT.md`. Keys live only in n8n's encrypted credential store.

## Quickstart, two ways to build it

### Option A, Import the ready-made workflow
1. In n8n, click **Add workflow → Import from File** (or **Import from URL**).
2. Import `workflow/huggingface-talent-radar.json` from this repo.
3. Connect your credentials and set your Airtable destination (see Setup details below).
4. Run a manual test, verify rows in Airtable, then activate.

### Option B, Rebuild it from a single prompt
1. Open n8n and create a new workflow.
2. Open the **n8n AI Assistant**.
3. Paste the full prompt from [`PROMPT.md`](PROMPT.md).
4. Connect credentials, test, and activate.

`PROMPT.md` describes every node so Claude / the n8n AI Assistant can reconstruct the workflow from scratch — handy if you want to understand or customize each step.

## Setup details

### 1) Import the workflow
In n8n, import `workflow/huggingface-talent-radar.json`.

### 2) Connect credentials
This template ships with **no credentials attached**. On import, connect your own:

- **Hugging Face** (`huggingFaceApi`), used by all Hugging Face HTTP Request nodes
- **Anthropic** (`anthropicApi`), used by **Score with Claude**
- **GitHub** (`githubApi`), used by **Find on GitHub**
- **Airtable** (`airtableTokenApi`), used by **Save to Airtable**

### 3) Point at your Airtable base
Open the **Save to Airtable** node and set your own destination:

- `YOUR_AIRTABLE_BASE_ID` → your Airtable base
- `YOUR_AIRTABLE_TABLE_ID` → your Airtable table

Or select them from the dropdowns once your Airtable credential is connected. Make sure your table includes the fields listed below.

### 4) Configure the role you're hiring for
Open **Score with Claude** and edit the `ROLE I AM HIRING FOR` line in the prompt. The default is:

> Senior ML Engineer. Must have: Python, PyTorch, LLM training or fine-tuning experience, 4+ years

Also update `source_role` in the **Parse Score** node so Airtable records the role you sourced for.

### 5) Customize your searches (optional)
The 4 search nodes use example seeds (`text-generation`, `transformers`, `EXAMPLE_ORG`). Edit the Search 1–4 URLs to match your hiring targets. Replace `EXAMPLE_ORG` in **Search 3 - By Org** with a real Hugging Face organisation.

### 6) Run a test
Execute one manual run, confirm records are written to Airtable, and review summary quality. The **Within Test Budget?** node caps Claude scoring to the first 2 candidates per run while testing.

### 7) Activate automation
Raise the limit in the **Within Test Budget?** node (e.g. to 20+) so full runs process, then turn on the workflow. It runs weekly on **Monday at 07:00** in your n8n instance timezone — change this in the **Every Monday 7am** node.

## Airtable fields

Fields the workflow **writes**:

- full_name
- hf_url
- github_url
- location_city
- source
- source_role
- date_sourced
- fit_score
- priority_action
- key_strengths
- key_gaps
- outreach_hook
- top_model_id
- top_model_downloads
- top_model_task
- num_models_published
- hf_followers
- hf_organisations
- estimated_seniority
- impact_signal

Optional **manual tracking** columns (not filled by the workflow — add them if you want to track outreach in Airtable):

- current_company
- contacted
- replied

## Customizing

- **Different task**, change `pipeline_tag` in the search URLs (e.g. `automatic-speech-recognition`, `image-classification`).
- **Different library**, change `library` in Search 4 (e.g. `diffusers`, `sentence-transformers`).
- **Model / cost**, change the `model` field in **Score with Claude**.
- **Scoring strictness**, tune the system prompt and the `fit_score` threshold in **Good Enough?**.

## Repository structure

```
workflow/
  huggingface-talent-radar.json
.github/
  workflows/
    gitleaks.yml
README.md
PROMPT.md
.gitleaks.toml
.gitignore
LICENSE
```

## Security notes

- This repo contains **no API keys or tokens**. Credentials live in n8n's encrypted credential store and are never exported to the workflow JSON.
- The exported workflow JSON has all `credentials` blocks and internal credential IDs removed.
- Airtable base and table IDs are placeholders (`YOUR_AIRTABLE_BASE_ID` / `YOUR_AIRTABLE_TABLE_ID`), no private destination is exposed.
- No real candidate data, names, emails, or scraped profiles are committed. The workflow only defines the process; candidate data lives in your private Airtable.
- The organisation seed in Search 3 is a placeholder (`EXAMPLE_ORG`), not a real sourcing target.
- A **Gitleaks** GitHub Action scans every push and pull request for accidentally committed secrets.
- Never commit a `.env` file or paste secrets into the workflow JSON. Keep keys in n8n credentials or a secrets manager only.


## License

MIT.
