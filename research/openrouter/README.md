OpenRouter Research
===================

Overview
--------
- OpenRouter is an aggregation layer that provides a unified API for dozens of frontier and open-source LLMs (Anthropic Claude, OpenAI GPT, Meta Llama, Google Gemini, Mistral, etc.).  
- Core value: standardized REST interface, dynamic model routing, transparent pricing, usage analytics, and built-in “ghostwriter” attribution back to model providers.  
- Endpoints are served from `https://openrouter.ai/api/v1`; auth uses bearer tokens generated in the OpenRouter dashboard.

Primary Use Cases
-----------------
1. **Model Marketplace Integration** – Give internal tools access to the newest models without rewriting adapters; swap models via the `model` parameter.  
2. **Cost/Quality Routing** – Implement cascading fallbacks (fast-cheap → slow-accurate) using the `provider` preferences and automatic retries.  
3. **Bring Your Own Key (BYOK)** – Forward requests through OpenRouter while billing still hits your own upstream provider keys, enabling global rate limiting and observability.  
4. **Safety Filters & Auditing** – Centralize moderation policies and user-level quotas, especially for consumer applications.  
5. **Revenue-Sharing Apps** – Use OpenRouter’s referral + creator programs to monetize AI apps while staying compliant with provider policies.

API Highlights
--------------
- **Chat Completions**: `POST /chat/completions` (OpenAI-compatible schema with `messages`, `model`, `temperature`, `max_tokens`, `route` hints).  
- **Image Generation**: select image-capable models (e.g., Stability, Playground) with similar payload shape to text requests.  
- **Streaming**: set `stream: true`; responses follow Server-Sent Events, which helps UI latency.  
- **Prompt Caching**: experimental header `X-Cache-Prompt` can reduce latency/cost for repeated prompts.  
- **Rate Limits**: default 60 RPM per key, but enterprise tiers allow custom SLAs; responses include `x-ratelimit-*` headers.

Implementation Notes
--------------------
- Map OpenRouter errors to retriable vs. fatal categories (`429` vs. `4xx/5xx`).  
- Persist `id` and `usage` fields for audit and cost dashboards.  
- Prefer JSON schema validation around `messages` to catch malformed roles before hitting the API.  
- When using BYOK, store provider key IDs separately from OpenRouter keys to rotate independently.  
- Respect the attribution requirement: display “Powered by OpenRouter” when re-selling model outputs unless you have an enterprise exemption.

Operational Checklist
---------------------
- Configure observability (Prometheus/Grafana or vendor) to monitor latency per model and fallback frequency.  
- Set per-user or per-team spending caps through OpenRouter dashboard automation hooks.  
- Maintain a regression suite with “golden prompts” to detect quality drift after changing routing rules.  
- For sensitive data, enable zero-retention mode or self-hosted proxy; encrypt payload logs at rest.

Example Request
---------------
```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "HTTP-Referer: https://yourapp.example" \
  -H "X-Title: My AI App" \
  -d '{
    "model": "anthropic/claude-3.5-sonnet",
    "messages": [
      {"role": "system", "content": "You are a concise support assistant."},
      {"role": "user", "content": "Summarize ticket #1234 in 4 bullet points."}
    ],
    "temperature": 0.2,
    "stream": true
  }'
```

KPIs & Success Metrics
----------------------
- Latency per model tier (P50/P95)  
- Cost per generated token relative to direct provider billing  
- Fallback activation counts (indicates provider instability)  
- Safety violation rate post-moderation  
- Revenue share earned vs. traffic driven (for public apps)

Sources & References
--------------------
- Official docs: <https://openrouter.ai/docs>  
- Pricing & model catalog: <https://openrouter.ai/models>  
- API compatibility notes: <https://openrouter.ai/docs#compatibility>  
- Security/retention FAQ: <https://openrouter.ai/faq>
