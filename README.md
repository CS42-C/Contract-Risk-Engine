# Contract-Risk-Engine
Built a contract risk scoring engine that reads commercial agreements and flags liability caps, IP traps, auto-renewal gotchas, and other clause-level risks. Rules-based for now, wired to accept an LLM layer when ready. HTML + vanilla JS, no dependencies.

# Contract Risk Engine

A browser-based tool that reads commercial contract text and produces a structured, clause-by-clause risk analysis. No backend, no dependencies, no API calls — just paste a contract and get a report in under two seconds.

Live demo: 

---

## Why I built this

Most people sign contracts without a structured view of what they're agreeing to. Legal review is expensive, slow, and often skipped entirely for routine commercial agreements. The clauses that tend to cause problems (liability caps set too low, auto-renewal windows buried in the fine print, one-sided indemnification) are almost always the same ones.

I wanted to see whether a lightweight, deterministic engine could surface those risks consistently, and what the UX should look like if the goal is clarity rather than completeness. This project is the answer to that question.

It also doubles as an architectural proof of concept. The rules engine is intentionally modular: swapping it out for an LLM call (Claude, GPT-4o, or a locally hosted model via Ollama) requires changing a single function. The rest of the application (the UI, the scoring display, the recommendations layer) stays untouched.

---

## What it analyses

The engine currently detects and scores eight clause types:

| Clause | What it looks for |
|---|---|
| Liability cap | Fixed cap amounts, asymmetric limits, caps below reasonable commercial thresholds |
| Termination rights | Immediate termination, termination for convenience, notice period adequacy |
| Auto-renewal | Opt-out notice windows, particularly those above 60 or 120 days |
| Payment terms | Payment periods above 45 days, punitive late payment interest rates |
| Intellectual property | Provider-retains-all structures, licence-only arrangements for bespoke work |
| Indemnification | One-sided obligations, gross negligence carve-outs limiting Provider exposure |
| Governing law & dispute resolution | Offshore jurisdictions, remote arbitration seats |
| Confidentiality | Third-party disclosure without prior consent |

Each clause is scored as low, medium, or high risk. The overall risk score (0–100) weights high-risk findings more heavily and feeds into a summary banner and a numbered recommendations list.

---

## Architecture

```
contract-risk-engine/
└── index.html        # everything: markup, styles, and logic in one file
```

Single-file by design. The goal was a tool that could be dropped anywhere, from GitHub Pages, a USB drive, to an email attachment, and work without a build step, a server, or an internet connection after the fonts load.

The analysis pipeline has three stages:

1. **Parsing** — the contract text is lowercased and searched for clause-type signals using regex patterns and keyword matching.
2. **Scoring** — each detected clause is evaluated against a risk rubric and assigned a tier (low / medium / high) based on specific conditions (e.g. auto-renewal notice period length, whether IP assignment goes to Client or Provider).
3. **Reporting** — clause findings, a weighted aggregate score, and a prioritised recommendations list are rendered into the UI.

### Upgrading to LLM-based analysis

The rules engine lives entirely inside `generateReport(text)`. To replace it with an LLM:

```javascript
async function generateReport(text) {
  const response = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      model: "claude-sonnet-4-20250514",
      max_tokens: 1000,
      messages: [{
        role: "user",
        content: `Analyse this contract and return a JSON object with fields: clauses (array of {type, risk, finding}), score (0-100), highCount, medCount, recs (array of strings), total.\n\nContract:\n${text}`
      }]
    })
  });
  const data = await response.json();
  return JSON.parse(data.content[0].text);
}
```

The rendering layer expects the same object shape regardless of where it comes from. For local/offline use, Ollama with Llama 3.1 or Mistral works with an identical function structure, just pointing at `http://localhost:11434/api/chat` instead.

---

## Running locally

No build step required. Clone the repo and open the file:

```bash
git clone https://github.com/CS42-C/contract-risk-engine.git
cd contract-risk-engine
open index.html   # or just drag it into a browser
```

---

## Limitations

This is a rules-based engine, not a trained model. It is good at catching common structural patterns in standard commercial contracts. It will miss things in heavily non-standard language, will not catch risks that require contextual reasoning across multiple clauses, and does not constitute legal advice. The LLM upgrade path exists precisely because these are real limitations worth solving.

---

## Roadmap

- LLM integration layer (Claude API) for semantic clause extraction
- PDF upload support
- Exportable PDF report
- Clause-level negotiation suggestions (not just risk flags)
- Support for Portuguese and Spanish contract language

---

## About

Built by [Francisco Costa] (https://www.linkedin.com/in/francscosta/) — Customs & Trade Compliance Analyst with a background in Civil Law and a focus on data analytics and business intelligence tooling.

The overlap between legal document structure and data modelling is something I find genuinely interesting. This project sits at that intersection.

Feedback and contributions welcome.
