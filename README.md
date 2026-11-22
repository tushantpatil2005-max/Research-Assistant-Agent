# AI Research Assistant Agent

**Recruitbots AI — Agentic AI Engineer · Challenge 1 (Beginner)**

One implementation file: [`research_assistant_agent.py`](research_assistant_agent.py).

An autonomous research agent that **reasons step by step**, **calls tools by name**, gathers evidence from multiple sources, and emits a **structured report with citations**. The control loop is iterative (thought → action → observation). There is **no fixed pipeline** of “always search then always fetch then always summarize.”

Customer-facing report text is free of machine-origin labels.

---

## Quick start

```bash
python research_assistant_agent.py --check

python research_assistant_agent.py -q "How do tool-using research agents combine retrieval, citations, and iterative reasoning?"

# Optional HTTP API
pip install fastapi uvicorn
python research_assistant_agent.py --serve
# POST /research  GET /health  GET /tools
```

Optional Llama for free-form next-action proposals (still falls back to the local planner):

```bash
python research_assistant_agent.py -q "..." --remote
```

---

## How the agent works

```text
User question
    │
    ▼
┌──────────────────────────────┐
│  Step loop (max 12)          │
│  1. Think about goal + state │
│  2. Choose a tool or finish  │
│  3. Observe tool result      │
│  4. Repeat until finish      │
└──────────────────────────────┘
    │
    ▼
Structured report + markdown + citations
```

Tool choice depends on **what is already in the observation history** (e.g. fetch only runs after search has returned ids; knowledge lookup runs when web evidence exists but method notes are still missing). That is iterative reasoning with tool calling, not a hardcoded workflow list.

---

## Success criteria

| Criterion | Implementation |
| --- | --- |
| At least 3 external tools | `web_search`, `fetch_source`, `knowledge_lookup`, plus `note_taking` |
| Structured reports | Title, executive summary, key findings, open questions, numbered references |
| Source citations | Each finding carries `citation_ids` mapped to references with title/URL |
| Ambiguous requests | Underspecified questions return `need_clarification` with one clear question |
| Context across steps | Session memory + `note_taking`; follow-ups can reference the prior report |

---

## Constraints

| Constraint | How it is met |
| --- | --- |
| Tool calling | Open registry with JSON schemas; agent emits `action` + `arguments` each step |
| No hardcoded workflow | Next tool is selected from current state (observations, gaps), not a fixed sequence |
| Iterative reasoning | Explicit thought/action/observation loop until `finish` or step budget |

---

## Tools

| Tool | Role |
| --- | --- |
| `web_search` | Keyword search over a curated web index (title, URL, snippet) |
| `fetch_source` | Load full text for a source id/URL from search |
| `knowledge_lookup` | Internal KB: ReAct notes, report schema, clarification policy |
| `note_taking` | Write/read working notes for multi-step context |

Schemas are exposed via `agent.tool_schemas()` and `GET /tools`.

---

## Report shape

```json
{
  "title": "...",
  "question": "...",
  "executive_summary": "...",
  "key_findings": [
    { "claim": "...", "citation_ids": [1], "source_id": "web_agent_tools" }
  ],
  "open_questions": ["..."],
  "references": [
    { "id": 1, "title": "...", "url": "https://..." }
  ]
}
```

Markdown rendering is available on `result["markdown"]` for export.

---

## Unmarked output

`strip_machine_labels` removes phrases such as “AI-generated”, “powered by AI”, or “as an AI…” from claims and markdown. Reports read as ordinary research notes.

---

## Self-check

`python research_assistant_agent.py --check` asserts:

- ≥3 registered tools  
- Ambiguous input → clarification, no fabricated topic  
- Full run uses ≥3 tools and returns citations  
- Search precedes fetch (state-dependent order)  
- Follow-up in the same session completes with memory  
- No machine-origin labels in the report body  

---

## File layout

```text
research_assistant_agent.py   # agent, tools, loop, report, API, self-check
README_research_agent.md      # this document
```
