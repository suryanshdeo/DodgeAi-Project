---
name: Dodge AI FDE Assignment - Project State
description: SAP O2C graph project state, stack, and phase progress for Dodge AI take-home task
type: project
---

Active take-home assignment for Dodge AI Forward Deployed Engineer role.

**Deadline:** 26th March 2026, 11:59 PM IST
**Submission form:** https://forms.gle/sPDBUvA45cUM3dyc8

**Project location:** `C:/Users/KIIT/Desktop/Surimon2/sap-o2c-graph/`
**Dataset location:** `C:/Users/KIIT/Desktop/Surimon2/sap-order-to-cash-dataset/sap-o2c-data/`
**Planning docs:** `C:/Users/KIIT/Desktop/Surimon2/PROJECT_PLAN.md` and `TASK_DOCUMENT.md`

**Tech stack:**
- Backend: Python + FastAPI + SQLite (o2c.db) + NetworkX
- LLM: OpenRouter free tier — `google/gemma-3-27b-it:free` (current, 27B, 131K ctx)
- Frontend: React + Vite + Cytoscape.js (inline styles, no Tailwind classes)
- Deploy: Render.com (single service, FastAPI serves React build)
- openai SDK version: 2.29.0 (upgraded from 1.54.0 to fix `proxies` kwarg error with httpx 0.28)

**Phase 1: COMPLETE + VERIFIED**
- `backend/ingest.py` — 19 tables ingested into o2c.db
- `backend/db.py` — execute_query(), get_schema(), get_connection()
- `backend/graph.py` — build_graph() → 669 nodes, 845 edges, Cytoscape.js format
- `backend/main.py` — GET /api/graph, GET /api/graph/node/{id}, GET /api/health
- Fix applied: `_parse_record()` in main.py deserializes nested JSON strings (e.g. creationTime)
- Server runs on port 8000

**Graph model:**
- 8 node types: SalesOrder (SO:), Delivery (DEL:), BillingDocument (BD:), JournalEntry (JE:), Payment (PAY:), Customer (CUST:), Product (PROD:), Plant (PLANT:)
- 7 edge types: PLACED_BY, HAS_PRODUCT, FULFILLED_BY, SHIPS_FROM, BILLED_AS, POSTED_TO, CLEARED_BY

**Phase 2 (Frontend - Graph UI): COMPLETE**
- `frontend/` — React + Vite scaffold
- `frontend/src/constants.js` — NODE_COLORS, NODE_TYPE_LABELS, CYTOSCAPE_STYLE
- `frontend/src/hooks/useGraph.js` — fetches /api/graph, injects colors
- `frontend/src/hooks/useNodeDetail.js` — fetches node detail, AbortController to cancel in-flight requests
- `frontend/src/components/GraphPanel.jsx` — Cytoscape.js canvas, cose layout; uses onNodeClickRef pattern to avoid listener accumulation bug
- `frontend/src/components/NodeSidebar.jsx` — slide-in panel with field rows, handles JE multi-records
- `frontend/src/components/Legend.jsx` — node type colors + node/edge counts
- `frontend/src/App.jsx` — header with "Ask AI" toggle, graph + sidebar + chat layout
- vite.config.js — proxy /api → localhost:8000, @tailwindcss/vite plugin
- Bug fixed: Cytoscape tap listeners accumulated on each render causing flickering sidebar. Fixed by storing onNodeClick in a ref and using empty deps [] on handleCyInit.

**Phase 3 (Chat + LLM pipeline): COMPLETE**
- `backend/llm.py` — guardrail_check(), nl_to_sql(), results_to_answer()
  - Uses python-dotenv to load `.env` from project root
  - MODEL constant: `google/gemma-3-27b-it:free`
  - Full schema string + 8 few-shot SQL examples in system prompt
- `POST /api/chat` in `backend/main.py` — 4-step: guardrail → NL→SQL → execute → answer
  - Returns: {question, allowed, sql, row_count, rows (first 20), answer}
- `frontend/src/components/ChatPanel.jsx` — chat UI with suggested starters, SQL collapsible, data table collapsible
- `.env` file at project root: OPENROUTER_API_KEY is set

**Working model:** `nvidia/nemotron-3-super-120b-a12b:free` (Nvidia provider, not Venice, no expiry)
- arcee-ai/trinity-large-preview:free was working but expires 31st March — replaced
- arcee-ai/trinity-mini:free — null content on guardrail (100 max_tokens too small for reasoning)
- Tried/failed: qwen3-coder, llama-3.3-70b, gemma-3-27b (Venice 429), gpt-oss-120b (data policy 404), stepfun/glm-4.5-air (null content — reasoning models eat all tokens)
- Kill server: `python -c "import subprocess; subprocess.run(['taskkill','/F','/PID',str(int(__import__('subprocess').run(['powershell','-c','(Get-NetTCPConnection -LocalPort 8000).OwningProcess'],capture_output=True,text=True).stdout.strip()))],capture_output=True)"`
- Start server: `cd C:/Users/KIIT/Desktop/Surimon2/sap-o2c-graph && python -m uvicorn backend.main:app --port 8000`
- Frontend dev: `cd frontend && npm run dev`

**Phase 4 (Polish + guardrails): NOT STARTED**
**Phase 5 (Deploy + README): NOT STARTED**
- Deploy: Render.com — FastAPI serves `frontend/dist/` as static files
- Need: start command, build command, static file mounting in main.py

**Why:** Dodge AI is an ERP automation platform. This task mirrors their actual product (graph = ERP entity model, NL query = how customers interact with their AI). They explicitly want AI coding session logs (Cursor, Claude Code).
