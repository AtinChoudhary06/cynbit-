# 🎯 Cynbit Lead Copilot
### AI Lead Qualification & Proposal Drafter

**🔗 Live demo: [cynbit-ohiuep2kwl5e8elsajm3nh.streamlit.app](https://cynbit-ohiuep2kwl5e8elsajm3nh.streamlit.app/)**

Proof of concept for the **Cynbit Technologies AI/ML Engineer Intern** assessment, by Atin Choudhary.

> Paste a client inquiry and get a category, lead score, extracted requirements,
> an indicative estimate and a ready-to-edit reply. A human always reviews before sending.

---

## The problem

Cynbit is a small AI and software services company (AI automation, custom software, SaaS, CRM
automation, cloud, SEO). Every inbound inquiry, whether from email, the contact form or a phone call,
has to be read, judged, scoped and answered by a person. For a small team this means:

- slow first responses, so warm leads go cold
- hours spent on low-quality or irrelevant inquiries (spam, job seekers, unrealistic budgets)
- inconsistent scoping and proposals
- engineers pulled away from delivery work

## The solution

| Step | What happens | Output |
|---|---|---|
| 1. **Classify** | Maps the inquiry to a Cynbit service, or marks it not a fit | `mobile_app`, `ai_automation`, ..., `other` |
| 2. **Extract** | Pulls out budget, timeline, features, industry, contact, missing info | Structured JSON |
| 3. **Score** | Explainable 0-100 score with a per-factor breakdown | Hot / Warm / Cold / Disqualified |
| 4. **Estimate** | Looks up a price and duration range from a pricing table | e.g. ₹2-8 lakh, 8-16 weeks |
| 5. **Draft** | Writes a personalised reply email using verified facts only | Editable email, human reviews |

### Example

**Input**
> Hi, I'm Rohit Mehra from StyleHub Retail Pvt Ltd, Delhi. We want a mobile app where customers can
> browse the catalog, add to cart, pay online and track their order. Live in 2 months, budget around 4 lakh.

**Output:** 🔥 Hot · 92/100 · `mobile_app` · indicative estimate ₹2-8 lakh, 8-16 weeks · reply email drafted

## Architecture

```
inquiry text
    │
    ▼
[1] Extraction ── LLM (Groq / Gemini / Claude) → strict JSON   (offline rules as fallback)
    │
    ▼
[2] Estimate ──── deterministic pricing table × complexity
    │
    ▼
[3] Scoring ───── deterministic, explainable rules (0-100)
    │
    ▼
[4] Drafting ──── LLM writes email from verified facts only    (template as fallback)
    │
    ▼
Streamlit UI → human reviews → send
```

### Key design decisions

- **The LLM only does language work** (reading and writing). Scoring and pricing are plain code, so
  results are explainable, tunable by the sales team, and the model can never invent a price or promise.
- **Graceful degradation.** If the API key is missing or the call fails, the app falls back to an
  offline rule-based engine and shows a warning, so the demo never crashes.
- **Human in the loop.** Output is a draft. Nothing is sent automatically.

## Scoring rubric

| Factor | Max | Logic |
|---|---|---|
| Service fit | 25 | Maps to a real Cynbit service |
| Budget | 25 | Stated and ≥ 90% of range minimum = full; below = partial; missing = 10 |
| Requirement clarity | 20 | 5 points per concrete feature extracted |
| Timeline | 15 | Stated and realistic = full; unrealistically short = 2; missing = 7 |
| Reachability | 15 | Contact details, company name, person name |

**Hot** ≥ 75 · **Warm** 50-74 · **Cold** < 50 · irrelevant or spam = **Disqualified**.

## Run locally

```bash
pip install -r requirements.txt
streamlit run app.py
```

The app works **with no API key** (offline demo mode). For LLM mode, set one key before starting:

```bash
# Windows cmd
set GROQ_API_KEY=your_key
# Windows PowerShell
$env:GROQ_API_KEY="your_key"
# Mac / Linux
export GROQ_API_KEY=your_key
```

Supported engines: **Groq** (`GROQ_API_KEY`), **Google Gemini** (`GEMINI_API_KEY`), **Anthropic Claude**
(`ANTHROPIC_API_KEY`). The first key found is used, and you can switch in the sidebar.
Optional model overrides: `GROQ_MODEL`, `GEMINI_MODEL`, `ANTHROPIC_MODEL`.

### Evaluate

```bash
python evaluate.py          # uses your key if set, otherwise offline mode
python evaluate.py demo     # force offline mode
```

## Deploy (Streamlit Community Cloud)

1. Push this repo to GitHub.
2. On [share.streamlit.io](https://share.streamlit.io), create an app pointing at `app.py`.
3. Under **Advanced settings → Secrets**, add `GROQ_API_KEY = "your_key"`.

Never commit API keys. `.env` is in `.gitignore`.

## Project structure

| File | Purpose |
|---|---|
| `app.py` | Streamlit UI |
| `leadengine.py` | Core pipeline: providers, extraction, scoring, pricing, drafting |
| `evaluate.py` | Runs the labelled samples and reports accuracy |
| `sample_inquiries.json` | 11 labelled test inquiries (good, vague, unrealistic, spam, job seeker) |
| `requirements.txt` | Python dependencies |
| `.env.example` | Environment variable template |

## Evaluation

Run `python evaluate.py` on the 11 labelled inquiries. Fill in your results:

| Mode | Category accuracy | Tier accuracy | Notes |
|---|---|---|---|
| Offline rules | 11/11 | 11/11 | Tuned on this same set, so this is not a generalisation claim |
| LLM (Groq) | _X/11_ | _X/11_ | _add after running_ |

## Limitations

- Offline mode uses keyword rules and was developed against the sample set, so its perfect score
  there does **not** show it will generalise. LLM mode should be judged on fresh inquiries.
- Pricing ranges are **illustrative assumptions** and must be replaced with Cynbit's real rate card.
- No CRM or email integration yet, no persistence or authentication.
- The draft reply must be reviewed by a human before sending.
- The shared demo uses a free-tier API key, so it may fall back to offline mode if the quota runs out.

## Roadmap

1. Replace the pricing table with real historical project data and learn estimates from past quotes.
2. Connect to the website form or inbox via webhook and push results into a CRM.
3. Retrieve relevant Cynbit case studies (RAG) so replies reference real past work.
4. Feedback loop: log whether a human edited the draft or won the deal, then recalibrate scores.
5. Guardrails: PII handling, prompt-injection filtering, audit log.

## Author

**Atin Choudhary** · Global Institute of Technology, Jaipur
Built for the Cynbit Technologies AI/ML Engineer Intern assessment.
