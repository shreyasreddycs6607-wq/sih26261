# PramaanAI (SIH26261, AICTE)

Decision-support tool: checks an institution's EoA application documents, verifies them with (mock) issuers, flags problems with evidence, scores risk, ranks inspections. **Read `BUILD_GUIDE.md` first** (features, stack, build order).

## Hard rules
- The software never approves, rejects or penalises. No "reject" button. A flag decision (ACCEPT/DISMISS) needs a written reason.
- Everything is synthetic. Every external system (SHCIL, DigiLocker, UDIN, fire/occupancy portals, AICTE records) is a MOCK and its responses carry `"mock": true`.
- Forensic flags say "Preliminary indicator - requires human verification" and are weighted by confidence.
- Norms live in `rulepacks/` YAML with clause + page, never hard-coded in Python.
- Quote the Handbook only after checking the official PDF (printed page N = PDF page N + 16). Paraphrase, don't copy.

## Layout and owners
| Path | Owner | What |
|---|---|---|
| `backend/` | Person 1 | FastAPI app, engines, pipeline |
| `mock_issuers/` | Person 1 | fake external systems, port 9000 |
| `rulepacks/`, `scripts/`, `data/` | Person 1 | rule YAML, synthetic data, proof scripts |
| `frontend/` | Person 2 | React + Vite, port 5173 |
| `contracts/openapi.json` | shared | generated, never hand-edited |
| `tests/` | whoever changes the thing | |

## Frozen contracts (change = its own commit starting `contract:` and both agree)
1. `rulepacks/schema.json` + pack layout
2. `backend/app/models.py` (19 tables)
3. `contracts/openapi.json` (10 routers under `/api/v1`)
4. `backend/app/schemas.py` (Flag, Fact, Verification, Risk ...)

After any API change: `python backend/scripts/export_openapi.py`, then `cd frontend && npm run gen:api`.

## Commands
```
pip install -r backend/requirements.txt
uvicorn app.main:app --reload --port 8000 --app-dir backend
uvicorn main:app --port 9000 --app-dir mock_issuers
cd frontend && npm install && npm run dev
pytest tests
```
Demo logins (stub only): `officer@pramaan.test`, `inst-a@pramaan.test`, `inspector@pramaan.test`, `admin@pramaan.test`; password `Pramaan@2026`.

## Style
Smallest working change. Engines are pure functions `engine(context) -> list[Flag]`. Add a test that fails first. Log decisions in `DECISIONS.md`.
