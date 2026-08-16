# Huggingface-Talent-Radar-with-Claude-n8n-Airtable
Automated Hugging Face talent sourcing with n8n + Claude + Airtable

![Hugging Face Talent Radar Workflow](HugginFace%20n8n%20Talent%20Radar.png)

# Huggingface-Talent-Radar-with-Claude-n8n-Airtable

Automated Hugging Face talent sourcing with n8n + Claude + Airtable

## Why Hugging Face is an underused sourcing channel for AI hiring

Hugging Face is where AI engineers and researchers publish the actual models they build: not papers about them and not social posts about them, but working model artifacts that developers use in production.

Download counts are a strong proxy for real world impact. If a model has hundreds of thousands of downloads, it suggests broad adoption and practical relevance beyond a CV bullet point.

You can typically discover three high value profile types:
Model authors: engineers who trained and published models
Space creators: engineers who built interactive AI demos
Organisation members: engineers associated with AI companies or research labs

This repository gives you a weekly n8n workflow that searches Hugging Face using four methods, evaluates profiles with Claude, and stores qualified candidates in Airtable with personalized outreach (optinal) hooks informed by model impact signals. 

## Who this is for (and not for)

**This is for you if:**
- You recruit ML/AI engineers and want a repeatable Hugging Face sourcing system.
- You want AI-assisted candidate scoring and personalized outreach hooks.
- You’re comfortable connecting APIs in n8n (Hugging Face, Claude, GitHub, Airtable).

**This is NOT for you if:**
- You only hire non-technical roles.
- You cannot use external APIs due to internal compliance/policy constraints.

## 5 accounts you need to create

1. n8n (n8n.io), the automation engine that connects everything  
2. Hugging Face (huggingface.co), your source of candidates  
3. Anthropic (console.anthropic.com), Claude AI scoring  
4. GitHub (github.com), author profile cross-reference  
5. Airtable (airtable.com), your candidate database  

## Airtable fields required by this workflow

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
