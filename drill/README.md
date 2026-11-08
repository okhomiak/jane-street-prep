# Drill: try-before-reveal question practice

[Index](../README.md) · [Question bank](../QUESTION-BANK.md)

A single-page practice app over the highest-value questions in this repo, built on the format that works best for interview prep: every item carries a **level** (L1 screen, L2 working, L3 senior, L4 staff), a **type** (concept, trade-off, estimate, design, code, incident, behavioural), a **time estimate**, a **short answer**, the **follow-ups** an interviewer reaches for, and the **wrong answer they hear most**. You answer out loud first, then reveal. Mark each as nailed or needs work; progress stays in your browser.

The format borrows the structure of [AI Engineering from Scratch's interview bank](https://aiengineeringfromscratch.com/interviews) (levels, item types, follow-ups, common wrong answers), applied to this loop. Its content is AI engineering, not security, so none of it is imported; the security items it does have (prompt injection, sandboxes, MCP gateways) are already covered more specifically in [block 16](../16-ai-and-llm-security/README.md).

## Use it

**Online, via GitHub Pages (recommended).** In the repo on GitHub: Settings → Pages → Build and deployment → Source "Deploy from a branch" → branch `main`, folder `/ (root)` → Save. After a minute, open:

```
https://okhomiak.github.io/jane-street-prep/drill/
```

**Locally.** Browsers block `fetch` from `file://`, so serve the folder:

```
cd drill && python3 -m http.server 8000
# then open http://localhost:8000/
```

## How to drill

- Start with **Jane Street confirmed only** ticked and work until everything is "nailed".
- Then filter by **level L3 and L4**; that is the depth this loop interviews at.
- Use **Random drill** daily for ten minutes: it picks from what you have not nailed.
- When a short answer surprises you, open the full answer in the block and read the whole section, not just the line.
- The **wrong answer** line is the most useful field. If your first instinct matched it, that is the thing to fix.

## Add or edit questions

Everything is in `questions.json`. Each item:

```json
{"id": "unique-id", "block": "07-windows-and-endpoints", "link": "../07-windows-and-endpoints/README.md",
 "js": false, "level": "L3", "type": "Concept", "time": 10,
 "q": "The question", "short": "One or two sentences.",
 "followups": ["Probe one", "Probe two"], "wrong": "The common bad answer."}
```

Keep short answers to two sentences; the block holds the full answer. Set `"js": true` only for questions reported in a Jane Street loop or your own online assessment.
