# Build prompt — Hugging Face Talent Radar

Paste this into the n8n AI Assistant to rebuild the workflow from scratch. Connect your own credentials when prompted — never paste API keys into the prompt.

---

Build an n8n workflow called "Hugging Face Talent Radar" that sources ML engineering candidates from Hugging Face, scores them with Claude, and saves qualified ones to Airtable.

**Trigger**
- A Schedule Trigger named "Every Monday 7am" that runs weekly on Monday at 07:00.

**Discovery — 4 parallel HTTP Request nodes (GET, Hugging Face API, `huggingFaceApi` credential, header `Accept: application/json`)**
1. "Search 1 - By Task": `https://huggingface.co/api/models?sort=downloads&direction=-1&limit=50&pipeline_tag=text-generation`
2. "Search 2 - Trending": `https://huggingface.co/api/models?sort=likes7d&direction=-1&limit=50&pipeline_tag=text-generation`
3. "Search 3 - By Org": `https://huggingface.co/api/models?author=EXAMPLE_ORG&sort=downloads&direction=-1&limit=50` (replace EXAMPLE_ORG with a real org)
4. "Search 4 - By Library": `https://huggingface.co/api/models?sort=downloads&direction=-1&limit=50&library=transformers&pipeline_tag=text-generation`

**Merge**
- A Merge node "Combine All Results" (mode: append, 4 inputs) collecting all four searches.

**Extract Unique Authors (Code node)**
- Read all merged models, reset a per-run static counter `hfCandidateCount = 0`, extract `author` (fallback `id.split('/')[0]`), deduplicate, sort by downloads desc, and return the top 80 as `{ hf_username, model_id, model_downloads, model_likes, model_task, model_last_updated }`.

**Process One By One (Split In Batches, batchSize 1)**
- Loop over candidates. The empty "done" output ends the loop; the loop output goes to profile lookup.

**Get HF Profile (HTTP Request, `huggingFaceApi`)**
- GET `https://huggingface.co/api/users/{{ $json.hf_username }}/overview`, header `Accept: application/json`.
- On error: continue (error output) — org accounts 404 here and are routed back to the batch loop, effectively skipping them.

**Find on GitHub (HTTP Request, `githubApi`)**
- GET `https://api.github.com/search/users?q={{ $json.fullname || $('Process One By One').first().json.hf_username }}&per_page=3`, header `Accept: application/vnd.github+json`.

**Format for Claude (Code node)**
- Combine HF profile, model data, and the first GitHub match. Increment static `hfCandidateCount`. Output identity, bio, location, model stats, GitHub cross-reference, and `candidate_number`.

**Within Test Budget? (IF node)**
- Keep only candidates where `candidate_number <= 2` (raise this for production). True → score; False → back to loop.

**Score with Claude (HTTP Request, `anthropicApi`)**
- POST `https://api.anthropic.com/v1/messages`, headers `anthropic-version: 2023-06-01` and `content-type: application/json`.
- Model `claude-sonnet-4-5-20250929`, max_tokens 700. System prompt: senior technical recruiter scoring HF profiles, return only valid JSON.
- User prompt includes a `ROLE I AM HIRING FOR:` line you edit, the profile fields, and asks for JSON with `fit_score`, `priority`, `key_strengths`, `key_gaps`, `outreach_hook`, `estimated_seniority`, `likely_employed`, `impact_signal`.

**Parse Score (Code node)**
- Strip code fences, `JSON.parse`, merge with the formatted profile, set `source: "Hugging Face"`, `source_role` (edit this), and `date_sourced` (today).

**Good Enough? (IF node)**
- Keep only `fit_score > 55`. True → save; False → back to loop.

**Save to Airtable (Airtable node, `airtableTokenApi`)**
- Create a record in base `YOUR_AIRTABLE_BASE_ID`, table `YOUR_AIRTABLE_TABLE_ID`, mapping the fields listed in the README. Then route back to "Process One By One".

Connect credentials for Hugging Face, Anthropic, GitHub, and Airtable when prompted. Do not hardcode any secrets.
