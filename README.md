# Huggingface-Talent-Radar-with-Claude-n8n-Airtable
Automated Hugging Face talent sourcing with n8n + Claude + Airtable

![Hugging Face Talent Radar Workflow](HugginFace%20n8n%20Talent%20Radar.png)

# Huggingface-Talent-Radar-with-Claude-n8n-Airtable

Automated Hugging Face talent sourcing with n8n + Claude + Airtable

## Who this is for (and not for)

**This is for you if:**
- You recruit ML/AI engineers and want a repeatable Hugging Face sourcing system.
- You want AI-assisted candidate scoring and personalized outreach hooks.
- You’re comfortable connecting APIs in n8n (Hugging Face, Claude, GitHub, Airtable).

**This is NOT for you if:**
- You need a zero-setup SaaS tool.
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
- current_company
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
- contacted
- replied
