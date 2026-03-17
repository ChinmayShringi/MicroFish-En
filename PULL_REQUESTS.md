# Pull Request History

This document records all pull requests from the original [MiroFish](https://github.com/666ghj/MiroFish) repository, translated to English.

> **Total PRs:** 88 | **Open:** 61 | **Merged:** 1 | **Closed:** 26

---

## Open PRs

### PR #233: fix: resource leaks and connection safety
- **Author:** @0xNyk
- **Branch:** `fix/resource-leaks-pr` -> `main`
- **Created:** 2026-03-17
- **Status:** Open
- **Labels:** *None*
- **Description:**

## Summary
- Use `with` context managers for all SQLite connections in simulation.py, simulation_runner.py, and simulation scripts
- Add `@after_this_request` temp file cleanup in report download endpoint
- Add `finally` block for file handle safety in simulation runner subprocess creation

## Test plan
- [x] Verify backend starts: `cd backend && uv run python -c "from app import create_app; create_app()"`
- [x] Test report download endpoint creates and cleans up temp files
- [x] Verify simulation scripts handle SQLite connections properly

---

### PR #232: fix: Docker security hardening
- **Author:** @0xNyk
- **Branch:** `fix/docker-hardening` -> `main`
- **Created:** 2026-03-17
- **Status:** Open
- **Labels:** *None*
- **Description:**

## Summary
- Multi-stage Dockerfile: separate build and runtime stages
- Run as non-root `mirofish` user
- Serve frontend with `serve` static server instead of Vite dev server
- Add `HEALTHCHECK` instruction to Dockerfile
- Add healthcheck config to `docker-compose.yml`
- Update `.dockerignore` with additional exclusions

## Test plan
- [x] `docker compose build` succeeds
- [x] Container starts and healthcheck passes
- [x] Frontend is served correctly on port 3000
- [x] Backend API responds on port 5001

---

### PR #231: fix: XSS prevention and frontend cleanup
- **Author:** @0xNyk
- **Branch:** `fix/frontend-xss-cleanup` -> `main`
- **Created:** 2026-03-17
- **Status:** Open
- **Labels:** *None*
- **Description:**

## Summary
- Add DOMPurify dependency and create `sanitize.js` utility
- Sanitize all `v-html` content through `renderMarkdown()` in Step4Report and Step5Interaction
- Guard all `setInterval` calls with `clearInterval` to prevent timer leaks in Step3Simulation, MainView, SimulationRunView

## Test plan
- [x] Verify frontend builds: `cd frontend && npm run build`
- [x] Test report and interaction views render markdown correctly
- [x] Verify no duplicate timers accumulate during navigation

---

### PR #230: fix: remove traceback exposure from API responses
- **Author:** @0xNyk
- **Branch:** `fix/remove-traceback-exposure` -> `main`
- **Created:** 2026-03-17
- **Status:** Open
- **Labels:** *None*
- **Description:**

## Summary
- Remove `traceback.format_exc()` from all 51 jsonify error responses across graph.py, simulation.py, report.py
- Log tracebacks server-side with `logger.exception()` instead
- Prevents leaking internal stack traces to API consumers

## Test plan
- [x] Verify backend starts: `cd backend && uv run python -c "from app import create_app; create_app()"`
- [x] Trigger API errors and verify responses no longer contain traceback field
- [x] Verify tracebacks still appear in server logs

---

### PR #229: fix: secure production defaults
- **Author:** @0xNyk
- **Branch:** `fix/secure-defaults` -> `main`
- **Created:** 2026-03-17
- **Status:** Open
- **Labels:** *None*
- **Description:**

## Summary
- Default `DEBUG` to `False` instead of `True`
- Generate random `SECRET_KEY` if not set (remove hardcoded fallback)
- Configure CORS origins from `CORS_ORIGINS` env var instead of wildcard `*`
- Default host to `127.0.0.1` instead of `0.0.0.0`
- Add security headers (X-Content-Type-Options, X-Frame-Options, X-XSS-Protection)

## Test plan
- [x] Verify backend starts with no env vars set
- [x] Verify CORS_ORIGINS env var is respected
- [x] Check response headers include security headers

---

### PR #228: fix: path traversal and input validation
- **Author:** @0xNyk
- **Branch:** `fix/path-traversal-input-validation` -> `main`
- **Created:** 2026-03-17
- **Status:** Open
- **Labels:** *None*
- **Description:**

## Summary
- Create `validators.py` with `validate_safe_id()` and `validate_path_within()` utilities
- Validate all user-supplied IDs (project_id, graph_id, simulation_id, report_id, task_id) in API route handlers to prevent path traversal
- Enforce allowed file extensions in `save_file_to_project()`
- Add base directory containment check in `file_parser.py`

## Test plan
- [x] Verify backend starts: `cd backend && uv run python -c "from app import create_app; create_app()"`
- [x] Test API with valid IDs returns normal responses
- [x] Test API with malicious IDs (e.g., `../../../etc/passwd`) returns 400 error

---

### PR #227: feat: add Mem0 memory provider support and update docs
- **Author:** @kartik-mem0
- **Branch:** `feat/mem0-integration` -> `main`
- **Created:** 2026-03-17
- **Status:** Open
- **Labels:** `documentation`, `enhancement`, `Memory Layer`, `size:XXL`
- **Description:**

Adds Mem0 as an optional memory backend alongside the existing Zep Cloud integration. Users choose via MEMORY_PROVIDER env var (zep default, mem0 optional). Supports both Mem0 Platform (SaaS) and
   Mem0 OSS (self-hosted). Zero breaking changes — all existing Zep behavior is untouched.

  How it works

  MEMORY_PROVIDER=zep (default) → existing Zep services, no change
  MEMORY_PROVIDER=mem0 → Mem0 backend (Platform or OSS)

  A thin MemoryProvider interface sits between API routes and the backend. ZepProvider delegates to existing Zep services (pure delegation, no behavior change). Mem0Provider implements the same
  interface using mem0ai SDK.

  Existing Zep service files (graph_builder.py, zep_entity_reader.py, zep_tools.py, zep_graph_memory_updater.py) are completely untouched.

  Setup for Mem0

  pip install mem0ai  # only needed if using Mem0

  Platform (managed)
  MEMORY_PROVIDER=mem0
  MEM0_API_KEY=your_key

  OR OSS (self-hosted)
  MEMORY_PROVIDER=mem0
  OPENAI_API_KEY=your_key

  Test plan

  - MEMORY_PROVIDER=zep (default) — full pipeline works unchanged
  - MEMORY_PROVIDER=mem0 + MEM0_API_KEY — Platform mode works
  - MEMORY_PROVIDER=mem0 + OPENAI_API_KEY — OSS mode works
  - No mem0ai installed + MEMORY_PROVIDER=zep — works fine (mem0ai is optional)

---

### PR #222: feat: integrate OpenRouter as a fallback LLM option; update configuration and client handling
- **Author:** @Hugo-SEQUIER
- **Branch:** `main` -> `main`
- **Created:** 2026-03-17
- **Status:** Open
- **Labels:** `enhancement`, `size:M`
- **Description:**

Add OpenRouter as a fallback LLM provider — when LLM_API_KEY is not set, the app automatically falls back to OPENROUTER_API_KEY
Auto-detect OpenRouter from the base URL and inject recommended HTTP-Referer / X-Title headers on all API calls

---

### PR #221: Update vite.config.js
- **Author:** @kashishgfuturetech-eng
- **Branch:** `patch-1` -> `main`
- **Created:** 2026-03-17
- **Status:** Open
- **Labels:** `size:XS`
- **Description:**

*No description provided.*

---

### PR #215: feat(frontend): add Vue I18n 9, locale files for home page
- **Author:** @isabelccc
- **Branch:** `feat/i18n-setup-home` -> `main`
- **Created:** 2026-03-17
- **Status:** Open
- **Labels:** `enhancement`, `size:L`
- **Description:**

feat(frontend): Add Vue I18n 9 and internationalize the Home page
# This PR is part 1 of the page internationalization series
- Introduced vue-i18n@9; added zh-CN.json and en.json under src/locales (covering home.*, common.* and other namespaces)
- Created src/i18n.js: uses createI18n, reads locale from localStorage, fallback to en, and implements applyLocaleToDocument()
- In main.js: app.use(i18n), applies current language to document on initialization
- Home.vue: replaced UI text with $t('home.xxx'); added language switch in navbar (Chinese / English); switching updates localStorage, document.documentElement.lang, and <title>
- Added docs/I18N-BRANCHES.md, describing branch planning for subsequent Process, History, Steps, Report, Interaction modules
For more details see /MiroFish/docs/I18N-BRANCHES.md
See reference screenshots below
Also added a language option in the top right corner


<img width="267" height="59" alt="Screenshot 2026-03-16 at 8 53 32 PM" src="https://github.com/user-attachments/assets/3875904b-2fb8-4ac1-9247-2bd8fb11c08d" />
<img width="560" height="585" alt="Screenshot 2026-03-16 at 8 53 26 PM" src="https://github.com/user-attachments/assets/f6cc9327-7e8c-47cb-8f37-9e1ea71ab69c" />



---

### PR #214: feat(i18n): add Korean locale support
- **Author:** @uppinote20
- **Branch:** `feature/i18n-korean` -> `main`
- **Created:** 2026-03-17
- **Status:** Open
- **Labels:** `enhancement`, `size:XXL`
- **Description:**

## Summary
- Add Korean locale to the i18n system introduced in #119
- All 241+ frontend translation keys covered across 15 sections
- 270+ backend message keys translated (API responses, logs, progress messages)
- LLM output language instructions for Korean (ontology, reports, profiles, simulation config)
- Korean README (README-KO.md) added

## Changes

### Frontend
- **New:** `frontend/src/i18n/locales/ko.js` — Full Korean translation
- **Modified:** `frontend/src/i18n/index.js` — Register Korean locale
- **Modified:** `frontend/src/components/LanguageSelector.vue` — Add Korean toggle (Chinese/English/Korean)
- **Modified:** All Vue views/components — addLog messages converted to i18n keys
- **Modified:** `frontend/index.html` — Add Noto Sans KR font

### Backend
- **Modified:** `backend/app/utils/error_messages.py` — 270+ keys with zh/en/ko translations
- **Modified:** `backend/app/utils/request_locale.py` — Accept `ko` locale
- **Modified:** `backend/app/services/ontology_generator.py` — Korean LLM language instruction
- **Modified:** `backend/app/services/report_agent.py` — Korean report/chat/progress messages
- **Modified:** `backend/app/services/oasis_profile_generator.py` — Korean profile generation
- **Modified:** `backend/app/services/simulation_config_generator.py` — Korean config generation
- **Modified:** All API routes — Localized response messages and logger output

### Docs
- **New:** `README-KO.md` — Korean README
- **Modified:** `README.md`, `README-EN.md` — Added Korean language link

## Test plan
- [x] Language selector shows 3 options: Chinese / English / Korean
- [x] Switching to Korean updates all UI text to Korean
- [x] Locale persists across page refresh (localStorage)
- [x] API requests include correct `X-Locale: ko` header
- [x] Backend error/log messages display in Korean
- [x] Ontology analysis generated in Korean
- [x] Report (title, summary, sections) generated in Korean
- [ ] Chat with ReportAgent responds in Korean
- [x] System dashboard logs di

*... (truncated)*

---

### PR #204: Harden ontology handling for Zep-backed simulations
- **Author:** @nativ3ai
- **Branch:** `fix/zep-ontology-sanitization` -> `main`
- **Created:** 2026-03-16
- **Status:** Open
- **Labels:** `size:XXL`
- **Description:**

## Summary
- normalize generated edge names to Zep-compatible `SCREAMING_SNAKE_CASE` before ontology submission
- skip malformed ontology rows and invalid attribute/source-target entries instead of crashing on missing keys
- allow the Vite dev server port to be overridden via `MIROFISH_FRONTEND_PORT`

## Why
Running MiroFish against Grok-generated ontologies exposed two failure modes in the graph pipeline:
1. mixed-case edge names such as `ANALYzes` are rejected by Zep
2. malformed entity or edge rows from the ontology LLM output can trigger `KeyError: 'name'`

These changes make the pipeline tolerate that noise and keep the simulation workflow moving.

## Validation
- `python3 -m py_compile backend/app/services/graph_builder.py backend/app/services/ontology_generator.py`
- verified the patched flow locally against Grok + Zep on real simulation runs


---

### PR #200: feat(backend): add Azure OpenAI provider and LLM provider switch
- **Author:** @gonewiththeway
- **Branch:** `feat/azure-openai-llm` -> `main`
- **Created:** 2026-03-15
- **Status:** Open
- **Labels:** `enhancement`, `LLM API`, `size:L`
- **Description:**

feat(backend): add Azure OpenAI provider and LLM provider switch

- Add LLM_PROVIDER (openai | azure_openai) and Azure env vars in config
- LLMClient: use Azure when provider=azure_openai (base_url, deployment, max_completion_tokens)
- Use max_completion_tokens for azure_openai to satisfy Azure API
- Add request/response and exception logging in llm_client for debugging
- Extend .env.example with LLM_PROVIDER and Azure OpenAI options
- Update README.md and README-EN.md with provider docs and env examples
- No new Python dependencies; pyproject.toml and uv.lock unchanged

Made with [Cursor](https://cursor.com)

---

### PR #198: feat: Add professional English translations and language toggle
- **Author:** @ajwise9
- **Branch:** `feature/i18n-english-translation` -> `main`
- **Created:** 2026-03-15
- **Status:** Open
- **Labels:** `enhancement`, `size:XXL`
- **Description:**

This PR introduces a professional English translation to the frontend UI. It adds a `src/i18n.js` file that centralizes the English and Chinese copy, and allows users to seamlessly hot-swap between languages using a toggle button in the navbar.

---

### PR #190: feat(xml): add XML file support for parsing and ontology generation
- **Author:** @Eskeeet
- **Branch:** `feat/xml-support-pr` -> `main`
- **Created:** 2026-03-15
- **Status:** Open
- **Labels:** `enhancement`, `size:L`
- **Description:**

## Summary

- Add XML parsing to `FileParser` with streaming support for MediaWiki/Wikipedia dumps and a generic XML fallback for all other XML files
- Add `.xml` to `ALLOWED_EXTENSIONS` in config
- Update frontend file accept filter and validation to include `.xml`
- Add unit tests covering generic XML, MediaWiki XML, and edge cases (12/12 passing)

## Test plan

- [x] Run `backend/tests/test_file_parser_xml.py` unit tests (12/12 passed)
- [x] Upload a generic `.xml` file via the UI and verify text is extracted correctly
- [x] Upload a MediaWiki XML dump and verify article titles and content are extracted
- [x] Verify unsupported file extensions are still rejected

🤖 Generated with [Claude Code](https://claude.com/claude-code)

---

### PR #189: feat(i18n): add internationalization with vue-i18n and translate codebase to English
- **Author:** @ysfbsf
- **Branch:** `feat/i18n-support` -> `main`
- **Created:** 2026-03-15
- **Status:** Open
- **Labels:** `enhancement`, `size:XXL`
- **Description:**

## Summary

- Add `vue-i18n` with runtime language switching (zh/en) using `$t()` translation keys across all frontend components and views
- Pass `language` parameter through backend APIs (ontology generation, simulation config, profile generation) for localized output
- Translate all Chinese comments, docstrings, error messages, log messages, and configuration descriptions to English across 43+ files (backend APIs, utilities, scripts, config/deploy files, and frontend)

## Test plan

- [ ] Verify frontend renders correctly in English (default)
- [ ] Verify frontend renders correctly in Chinese when locale is switched
- [ ] Test ontology generation with `language=en` and `language=zh`
- [ ] Test simulation preparation with both language options
- [ ] Verify backend API error messages return in English
- [ ] Run existing test suite to ensure no regressions

---

### PR #188: Fix #133: Add root route to fix 404 error on backend root URL
- **Author:** @hkc5
- **Branch:** `main` -> `main`
- **Created:** 2026-03-14
- **Status:** Open
- **Labels:** `size:M`
- **Description:**

## Problem
When accessing the backend at `0.0.0.0:5001`, users receive a 404 error. This is confusing because the server appears to start successfully but the root URL returns nothing.

## Solution
Added a root route (`/`) that returns:
- Service name and status
- Available endpoints information
- API route prefixes

## Changes
- Modified `backend/app/__init__.py` to add an `@app.route('/')` endpoint

## Testing
The root endpoint now returns:
```json
{
  "service": "MiroFish Backend",
  "status": "running",
  "version": "1.0.0",
  "endpoints": {
    "health": "/health",
    "api": {
      "graph": "/api/graph",
      "simulation": "/api/simulation",
      "report": "/api/report"
    }
  }
}
```

Fixes #133

---

### PR #186: Fix #180: Handle unsupported response_format parameter for non-OpenAI providers
- **Author:** @hkc5
- **Branch:** `fix-180-response-format` -> `main`
- **Created:** 2026-03-14
- **Status:** Open
- **Labels:** `LLM API`, `size:L`
- **Description:**

## Problem
Ontology generation fails with 500 error when using LLM providers like Groq that don't support the OpenAI-specific `response_format={"type": "json_object"}` parameter.

## Root Cause
The code uses `response_format={"type": "json_object"}` which is an OpenAI-specific feature. When using Groq or other providers that don't support this parameter, the API call fails with a 500 error.

## Solution
Added graceful fallback logic in three files:
1. **llm_client.py** - Core LLM client with automatic retry without response_format
2. **simulation_config_generator.py** - Direct API calls with fallback
3. **oasis_profile_generator.py** - Direct API calls with fallback

When a 400/500 error related to response_format is detected, the code automatically retries without the parameter and relies on the system prompt to enforce JSON output format.

## Testing
- Tested with Groq API (llama-3.1-8b-instant, llama-3.2-1b-preview, llama-3.1-70b-versatile)
- Backwards compatible with OpenAI API
- No breaking changes to existing functionality

Fixes #180

---

### PR #179: fix: Standardize ontology entity and relationship names to PascalCase to support Zep graph building (fixes #178)
- **Author:** @i-icc
- **Branch:** `fix/issue178-ontology-pascalcase-for-zep` -> `main`
- **Created:** 2026-03-13
- **Status:** Open
- **Labels:** `size:M`
- **Description:**

## Description
Standardize ontology entity and relationship names to PascalCase format to support Zep graph building.

### Problem
Zep requires PascalCase naming for entities and relationships, but the ontology generator was producing names in various formats (snake_case, Chinese characters, etc.), causing failures when building graphs.

### Solution
- Added a name normalization step in the ontology generation pipeline
- Converts all entity and relationship names to PascalCase
- Handles Chinese character names by transliterating first
- Added validation to reject names that don't conform

---

### PR #176: conversion to english
- **Author:** @arshmakker
- **Branch:** `main` -> `main`
- **Created:** 2026-03-13
- **Status:** Open
- **Labels:** `documentation`, `size:XXL`
- **Description:**

*No description provided.*

---

### PR #168: fix: auto-detect platform to prevent silent data loss in Twitter-only simulations
- **Author:** @JasonOA888
- **Branch:** `fix/platform-auto-detect` -> `main`
- **Created:** 2026-03-13
- **Status:** Open
- **Labels:** `size:M`
- **Description:**

Fixes #150

## Problem
When a simulation is created with Twitter-only configuration (`enable_reddit=false`), all data retrieval APIs silently return empty results because they default to looking up Reddit data.

## Root Cause
- Platform parameter hardcoded to `'reddit'` in 3+ API locations
- Twitter-only simulations never create `reddit_simulation.db`
- APIs look for Reddit data → empty results → user sees no data

## Solution
1. Added `detect_platform_from_simulation()` - checks which db files exist
2. Added `get_platform_with_fallback()` - intelligent fallback
3. Updated 3 API endpoints to use auto-detection

## Affected Endpoints
- GET `/<simulation_id>/profiles`
- GET `/<simulation_id>/profiles/realtime`
- GET `/<simulation_id>/posts`

## Logic
- Check for actual db files: `reddit_simulation.db` and `twitter_simulation.db`
- Return platform that actually has data
- Fallback to `reddit` for backward compatibility

## Testing
- [x] Twitter-only simulation returns Twitter data (not empty)
- [x] Reddit-only simulation still works
- [x] Both-platform simulation defaults to Reddit

## Impact
Eliminates silent data loss for Twitter-only simulations

---

### PR #165: fix: Frontend API request failure and slow/frozen build during Docker deployment
- **Author:** @WinJayX
- **Branch:** `fix-api-baseURL` -> `main`
- **Created:** 2026-03-12
- **Status:** Open
- **Labels:** `size:S`
- **Description:**

## Description
Fix frontend API request failure and slow/frozen build issues during Docker deployment.

### Problem
1. Frontend API base URL was hardcoded to localhost, causing requests to fail in Docker
2. Frontend build process was extremely slow and sometimes froze in Docker containers

### Solution
- Made API base URL configurable via environment variables
- Optimized Docker build configuration for faster frontend builds

---

### PR #162: feat: add MiniMax model compatibility support
- **Author:** @octo-patch
- **Branch:** `feature/add-minimax-provider` -> `main`
- **Created:** 2026-03-12
- **Status:** Open
- **Labels:** `documentation`, `LLM API`, `size:L`
- **Description:**

## Summary

- Add automatic MiniMax model detection via model name and base URL
- Handle MiniMax API constraints: clamp `temperature` to `0.01` when set to `0` (MiniMax requires `temperature ∈ (0.0, 1.0]`), and replace unsupported `response_format` with prompt-based JSON instruction injection
- Strip `<think>` tags from MiniMax M2.5 model responses in `simulation_config_generator.py` and `oasis_profile_generator.py`
- Add MiniMax configuration examples to `.env.example` and documentation to `README.md` / `README-EN.md`
- Add 23 unit tests for all MiniMax compatibility functions

## Changes

| File | Description |
|------|-------------|
| `backend/app/utils/llm_client.py` | Add `_is_minimax()`, `_clamp_temperature()`, `_inject_json_instruction()` helpers; update `LLMClient.chat()` to conditionally skip `response_format` for MiniMax |
| `backend/app/services/simulation_config_generator.py` | Use MiniMax-aware JSON parsing and `<think>` tag stripping in `_call_llm_with_retry()` |
| `backend/app/services/oasis_profile_generator.py` | Use MiniMax-aware JSON parsing and `<think>` tag stripping in profile generation |
| `.env.example` | Add commented MiniMax configuration section |
| `README.md` | Add collapsible MiniMax setup guide (Chinese) |
| `README-EN.md` | Add collapsible MiniMax setup guide (English) |
| `backend/tests/test_minimax_compat.py` | 23 unit tests covering detection, temperature clamping, JSON injection, and response parsing |

## Test plan

- [x] All 23 unit tests pass (`pytest backend/tests/test_minimax_compat.py`)
- [ ] Verify MiniMax M2.5 model works end-to-end with simulation (requires MiniMax API key)
- [ ] Verify existing non-MiniMax models (e.g., qwen-plus) are unaffected

## MiniMax Configuration

```env
LLM_API_KEY=your_minimax_api_key
LLM_BASE_URL=https://api.minimax.io/v1
LLM_MODEL_NAME=MiniMax-M2.5
```

Supported models: `MiniMax-M2.5` (flagship, 204K context) and `MiniMax-M2.5-highspeed` (faster variant).


---

### PR #161: fix: normalize LLM output types in profile serialization
- **Author:** @ygh1254
- **Branch:** `fix/profile-type-normalization` -> `main`
- **Created:** 2026-03-12
- **Status:** Open
- **Labels:** `size:M`
- **Description:**

## Summary

- **Add type coercion helpers** (`_coerce_to_str`, `_coerce_to_str_list`) to safely convert `dict`/`list` LLM outputs into plain strings
- **Normalize fields at construction time** via `OasisAgentProfile.__post_init__` — covers `bio`, `persona`, `country`, `profession`, `gender`, `interested_topics`
- **Defensive coercion in serializers** (`_save_twitter_csv`, `_save_reddit_json`) as a second safety net before string operations like `[:150]` and `.replace()`
- **Normalize in `_generate_profile_with_llm()`** immediately after `json.loads()` to catch structured outputs before they reach the dataclass

## Root Cause

LLM JSON parsing (`json.loads`) can produce `dict` or `list` values for fields declared as `str` (e.g. `bio`, `persona`, `country`). The serialization code then crashes on string operations like slicing (`[:150]`) or `.replace()` applied to non-string types.

Traceback from issue:
```
File "backend/app/services/oasis_profile_generator.py", line 1296, in _save_reddit_json
  "bio": profile.bio[:150] if profile.bio else f"{profile.name}",
KeyError: slice(None, 150, None)
```

## Testing

The coercion logic handles:
- `bio` as `dict` → extracts text via common keys or `json.dumps`
- `persona` as `dict` → same
- `country` as `list` → joins into comma-separated string
- `interested_topics` as nested dicts → flattens to `list[str]`

Closes #154

---

### PR #155: chore: backend, frontend, i18n (en/zh), and Docker updates
- **Author:** @kaeli-byte
- **Branch:** `english-trans` -> `main`
- **Created:** 2026-03-12
- **Status:** Open
- **Labels:** `size:XXL`
- **Description:**

Made-with: Cursor

---

### PR #151: Fix silent data loss when platform defaults to reddit for Twitter-only simulations
- **Author:** @karesansui-u
- **Branch:** `fix/platform-default-reddit-silent-failure` -> `main`
- **Created:** 2026-03-11
- **Status:** Open
- **Labels:** `size:M`
- **Description:**

## Summary

- API retrieval endpoints (`/profiles`, `/profiles/realtime`, `/posts`, `/comments`) hardcoded `'reddit'` as the default platform
- When a Twitter-only simulation was run (`enable_reddit=false`), these APIs silently returned empty results because they looked for `reddit_simulation.db` / `reddit_profiles.json` which did not exist
- Frontend also hardcoded `'reddit'` in Vue components and API wrapper functions

## Changes

**Backend** (`simulation.py`, `simulation_manager.py`):
- Add `SimulationState.get_default_platform()` that reads `enable_twitter`/`enable_reddit` config
- Add `_get_default_platform()` helper in the API layer (follows existing `_check_simulation_prepared` pattern)
- Replace hardcoded `'reddit'` defaults in 4 endpoints with config-aware defaults
- Add `platform` parameter to `/comments` endpoint (was hardcoded to `reddit_simulation.db`)

**Frontend** (`simulation.js`, `Step2EnvSetup.vue`, `Step5Interaction.vue`):
- Remove hardcoded `'reddit'` defaults from 3 API functions; omit platform param to let backend decide
- Remove hardcoded `'reddit'` from 2 Vue component calls

**Scope**: This PR fixes the API retrieval layer only. The profile generation layer (`oasis_profile_generator.py`) already correctly selects platform based on simulation config.

**Backward compatible**: Explicit `platform=reddit` or `platform=twitter` still works as before.

## Test plan

- [x] Create a Twitter-only simulation (`enable_reddit=false`) and verify profiles/posts/comments are returned
- [x] Create a Reddit-only simulation and verify existing behavior is unchanged
- [x] Create a parallel (both platforms) simulation and verify default falls back to reddit

Fixes #150

---

### PR #144: feat(kg): add dual-mode knowledge graph support
- **Author:** @huamingjie0815
- **Branch:** `feat/local-knowledge-graph` -> `main`
- **Created:** 2026-03-11
- **Status:** Open
- **Labels:** `Memory Layer`, `size:XXL`
- **Description:**

## Summary
- Add kg_adapter for dual-mode knowledge graph (cloud/local)
- Support switching between Zep Cloud and local Graphiti + Neo4j
- Improve entity extraction and report agent robustness
- Add test_kg_adapter.py with unit tests

## Test plan
- [x] Test cloud mode with Zep Cloud
- [x] Test local mode with Graphiti + Neo4j
- [x] Run unit tests

🤖 Generated with [Claude Code](https://claude.com/claude-code)

---

### PR #143: docs: fix README alt text URL encoding
- **Author:** @fishwww-ww
- **Branch:** `docs/urlEncoding` -> `main`
- **Created:** 2026-03-11
- **Status:** Open
- **Labels:** `documentation`, `size:XS`
- **Description:**

## Summary

  Fix the Shanda image alt text in README.md by changing 666ghj%2MiroFish to 666ghj%2FMiroFish.

  ## Details

  666ghj%2MiroFish is not a valid URL-encoded representation, so it cannot be decoded correctly.
  Using 666ghj%2FMiroFish correctly encodes the slash and can be properly decoded to 666ghj/
  MiroFish.

  ## Impact

  Documentation-only change. No code or runtime behavior is affected.

---

### PR #141: feat: add entity deduplication after graph building
- **Author:** @Stayfoool
- **Branch:** `feature/entity-deduplication` -> `main`
- **Created:** 2026-03-11
- **Status:** Open
- **Labels:** `enhancement`, `size:XL`
- **Description:**

Hi @666ghj 

I noticed that during graph building, Zep sometimes creates duplicate 
entity nodes for the same real-world entity (e.g. "Trump" and "US President Trump"
appear as separate nodes). This affects the accuracy of the knowledge graph.

This PR adds an automatic entity deduplication step after graph building, 
using name similarity pre-filtering + type compatibility check + LLM 
confirmation to identify and merge duplicates.

Would appreciate it if you could take a look when you have time. 
Happy to make any changes based on your feedback. Thanks!

## Summary
- Add entity deduplication service that identifies and merges duplicate 
  nodes in the knowledge graph after building (e.g. "Trump" vs "US President Trump")
- When merging duplicate nodes, migrates all edges from removed nodes 
  to the primary node before deletion, preserving graph connectivity
- Three-layer filtering: name similarity pre-filter → type compatibility 
  check → LLM confirmation
- Integrate into graph build pipeline automatically (80%-90% progress stage)
- Add standalone POST /api/graph/deduplicate endpoint for manual dedup
- Display dedup report in frontend showing which entities were merged
<img width="755" height="277" alt="duplicate remove" src="https://github.com/user-attachments/assets/b405a413-315a-470b-b857-51dcada5d532" />


## Changes
- **New**: `backend/app/services/entity_deduplicator.py`
- **Modified**: `backend/app/api/graph.py`
- **Modified**: `backend/app/services/__init__.py`
- **Modified**: `frontend/src/views/Process.vue`
- **Modified**: `frontend/src/views/MainView.vue`
- **Modified**: `frontend/src/components/Step1GraphBuild.vue`

Closes #145

---

### PR #132: docs:add simple system architecture part for README-EN.md & README.md
- **Author:** @Noblegasesgoo
- **Branch:** `docs/add-sys-architecture-part` -> `main`
- **Created:** 2026-03-11
- **Status:** Open
- **Labels:** `documentation`, `size:L`
- **Description:**

## PR Title
docs(readme): simplify system architecture section to Layer Breakdown + Project Code Structure Tree only

## Summary
This PR simplifies the **System Architecture** section in both Chinese and English README files by keeping only two high-signal sections:

- **Layer Breakdown**
- **Project Code Structure Tree**

The previously added overall architecture diagram and related agent-intro block were removed to keep the documentation focused and maintainable.

## Changes
- Updated Chinese README: [README.md](/Users/noblegasesgoo/Documents/noblegasesgoo/ai/MiroFish/README.md)
- Updated English README: [README-EN.md](/Users/noblegasesgoo/Documents/noblegasesgoo/ai/MiroFish/README-EN.md)

### Kept
- Layered architecture table (responsibility-oriented)
- File-level project structure tree (code-oriented)

### Removed
- Overall architecture image module
- Associated architecture-intro text block

## Why
- Reduce visual/maintenance overhead in README
- Keep architecture docs concise and implementation-aligned
- Improve readability for contributors who want quick codebase orientation

## Impact
- Documentation-only change
- No runtime or API behavior changes

## Validation
- Verified both README files render correctly
- Confirmed section structure is consistent in CN/EN docs

---

### PR #131: feat(graph_builder): add retry mechanism for Zep Cloud connection failures
- **Author:** @EuanTop
- **Branch:** `feat/zep-retry-mechanism` -> `main`
- **Created:** 2026-03-11
- **Status:** Open
- **Labels:** `enhancement`, `size:L`
- **Description:**

## Description

Adds automatic retry mechanism to handle transient network errors when connecting to Zep Cloud API. This prevents graph build failures caused by temporary connection issues such as "Connection reset by peer" (errno 54).

The retry logic uses exponential backoff (2s, 4s, 6s) and provides detailed progress feedback to users.

## Changes

- Added retry logic (max 3 attempts) to `create_graph()` method in `graph_builder.py`
- Added retry logic to `set_ontology()` method for robust ontology configuration
- Added retry logic to `add_text_batches()` method for batch data uploads
- Implemented exponential backoff strategy with detailed console logging
- Added progress messages showing retry attempts and wait times

## Type of Change

- [ ] Bug fix
- [x] New feature
- [ ] Breaking change
- [ ] Documentation

## Testing

- Manually tested with unstable network conditions
- Verified retry mechanism triggers on connection failures
- Confirmed successful graph creation after transient network errors
- Tested with various network interruption scenarios

## Related Issue

Related to #60 - Improves robustness when connecting to Zep Cloud API

## Additional Notes

This change is backward compatible and doesn't modify the API interface. The retry mechanism is transparent to users and only activates when network errors occur.


---

### PR #129: fix(report_agent): handle API token overflow crash with context lengt…
- **Author:** @ai-x-builder
- **Branch:** `fix/fix-priority-issues-mNNjT` -> `main`
- **Created:** 2026-03-11
- **Status:** Open
- **Labels:** `LLM API`, `size:M`
- **Description:**

Add error handling in LLMClient for context_length_exceeded errors with
automatic message trimming and retry (fixes https://github.com/666ghj/MiroFish/issues/52)
Add configurable LLM_MAX_TOKENS env variable (default 4096) so users
with different models can set appropriate limits
Add message history pruning in report agent ReACT loop to prevent
unbounded context growth that causes token overflow
Import and handle openai.BadRequestError and APIError explicitly

---

### PR #126: feat: Add custom exceptions and enhanced config validation
- **Author:** @ZaviQ7
- **Branch:** `feature/custom-exceptions-and-config-validation` -> `main`
- **Created:** 2026-03-10
- **Status:** Open
- **Labels:** `enhancement`, `size:XXL`
- **Description:**

## Summary
This PR improves the robustness of the MiroFish backend by implementing two key architectural improvements:

### 1. Custom Exception Hierarchy
- Created a `MiroFishError` base class with error codes, severity levels, and HTTP status codes.
- Added domain-specific exceptions for Configuration, Graphs, Simulations, and External APIs to replace generic Exception catches.

### 2. Enhanced Configuration Validation
- Implemented `Config.validate_comprehensive()` to check for valid URL formats, numeric ranges, and placeholder API keys.
- Added directory permission checks and a formatted startup validation report.

## Testing
- Created unit tests in `backend/tests/test_exceptions.py` and `backend/tests/test_config_validated.py`.
- Verified all 15+ test cases pass locally.
- Ensured 100% backward compatibility with existing code.

---

### PR #125: fix: improve new-project network error diagnostics
- **Author:** @SergioChan
- **Branch:** `fix/issue-121` -> `main`
- **Created:** 2026-03-10
- **Status:** Open
- **Labels:** `enhancement`, `size:S`
- **Description:**

## Summary

Improve frontend error feedback when creating a new project so users can quickly diagnose "Network Error" and timeout failures instead of seeing a generic message.

## Changes

- Added `formatProjectInitError` in `frontend/src/views/Process.vue`
- Distinguish timeout errors and provide actionable hint (reduce file size / check model speed)
- Distinguish network errors and show configured backend URL (`VITE_API_BASE_URL` fallback)
- Surface backend `error/message` payload when available

## Testing

- Manual code-path review for `handleNewProject` catch block
- No automated frontend test harness in repo for this view

Fixes 666ghj/MiroFish#121

---

### PR #124: fix: robust JSON extraction for mixed LLM responses
- **Author:** @SergioChan
- **Branch:** `fix/issue-64` -> `main`
- **Created:** 2026-03-10
- **Status:** Open
- **Labels:** `LLM API`, `size:M`
- **Description:**

## SummarynnHarden backend JSON parsing for LLM responses so mixed outputs (markdown fences, pre/post text) are handled more robustly, reducing 500 errors reported during ontology generation.nn## Changesnn- Updated `LLMClient.chat()` to remove `<think ...>...</think>` tags case-insensitivelyn- Added `LLMClient._extract_json_payload()` to normalize and extract JSON from noisy model responsesn- Updated `chat_json()` to parse extracted payload instead of raw contentn- Added unit tests in `backend/tests/test_llm_client_json_extract.py` for fenced JSON and mixed-text extractionnn## Testingnn- Added targeted unit tests for extraction behaviorn- Could not execute tests in this environment because backend dev dependencies are not installed (`pytest`, `flask` missing)nnFixes 666ghj/MiroFish#64

---

### PR #122: fix(llm_client): remove response_format json_object for local LLM compatibility
- **Author:** @ImL1s
- **Branch:** `fix/lm-studio-json-object-compat` -> `main`
- **Created:** 2026-03-10
- **Status:** Open
- **Labels:** `LLM API`, `size:XS`
- **Description:**

## Problem

`chat_json()` uses `response_format={"type": "json_object"}`, but LM Studio and Ollama do not support this parameter (only `json_schema` or `text`), causing API calls to fail when using local LLMs.

Related references:
- LM Studio: https://github.com/lmstudio-ai/lmstudio-bug-tracker/issues/534
- Similar to issue #110 (API call failures)

## Solution

Remove `response_format` from `chat_json()`. The method already has robust markdown code fence cleanup logic (L93-97) that correctly parses JSON from raw LLM output, making `response_format` unnecessary.

This follows the same approach as commit 985f89f (handling `<think>` tags from reasoning models) — improving compatibility with diverse model outputs.

## Changed Files

- `backend/app/utils/llm_client.py`: Remove `response_format` in `chat_json()`, rely on prompt instructions + markdown cleanup

## Testing

- LM Studio + qwen3.5-9b: Full prediction pipeline (ontology → graph → prepare → simulate → report) passes
- Does not affect OpenAI API users (removing response_format is backwards compatible)

---

### PR #119: feat: add an option to switch to english language
- **Author:** @Pratiyankkumar
- **Branch:** `language-option` -> `main`
- **Created:** 2026-03-10
- **Status:** Open
- **Labels:** `enhancement`, `size:XXL`
- **Description:**

Right now the content of the website is mostly in Chinese , Added an button to switch between Chinese and english language .

[`Demo Video`](https://drive.google.com/file/d/15VYI0J1SoDRf27Zvprm1P-D4MO8hA7yE/view?usp=sharing)

---

### PR #118: feat(ragflow): add RAGflow as alternative graph backend with full pip…
- **Author:** @pratyush618
- **Branch:** `fix/ragflow-pattern-compliance` -> `main`
- **Created:** 2026-03-10
- **Status:** Open
- **Labels:** `documentation`, `enhancement`, `size:XXL`
- **Description:**

…eline compliance

- Add RagflowGraphBuilderService and RagflowEntityReader for self-hosted graph support
- Add _get_entity_reader() helper in simulation.py to auto-select reader by graph_id prefix
- Fix 4 simulation endpoints (get_graph_entities, get_entity_detail, get_entities_by_type, generate_profiles) to support ragflow_ graph IDs
- Guard ZepGraphMemoryManager.create_updater() to skip for RAGflow graph IDs
- Add get_node_edges, get_entity_with_context, get_entities_by_type methods to RagflowEntityReader
- Update config.py, project.py, graph.py, simulation_manager.py for dual-backend support
- Document RAGflow config in .env.example, README.md, and README-EN.md

---

### PR #116: Upgrade GitHub Actions
- **Author:** @ailuntz
- **Branch:** `chore/upgrade-actions` -> `main`
- **Created:** 2026-03-10
- **Status:** Open
- **Labels:** `size:XS`
- **Description:**

Fixes #92.\n\nBump checkout and build-push to latest major versions.

---

### PR #115: Use SPDX license string
- **Author:** @ailuntz
- **Branch:** `fix/pyproject-license` -> `main`
- **Created:** 2026-03-10
- **Status:** Open
- **Labels:** `size:XS`
- **Description:**

Fixes #46.\n\nSwitch project.license to SPDX string to avoid the deprecation warning.

---

### PR #114: Fix API base URL fallback
- **Author:** @ailuntz
- **Branch:** `fix/api-baseurl-default` -> `main`
- **Created:** 2026-03-10
- **Status:** Open
- **Labels:** `size:S`
- **Description:**

Fixes #93.\n\nUse VITE_API_BASE_URL when set; otherwise default to current origin.

---

### PR #113: docs(readme): add Japanese README
- **Author:** @eltociear
- **Branch:** `add-ja-doc` -> `main`
- **Created:** 2026-03-10
- **Status:** Open
- **Labels:** `documentation`, `size:L`
- **Description:**

I created Japanese translated README.

---

### PR #112: Korean README.md added
- **Author:** @waitle
- **Branch:** `main` -> `main`
- **Created:** 2026-03-10
- **Status:** Open
- **Labels:** `documentation`, `size:L`
- **Description:**

AI translated Korean readme file

---

### PR #108: feat(installer): add Windows installer build scripts
- **Author:** @JasonOA888
- **Branch:** `feat/windows-installer` -> `main`
- **Created:** 2026-03-09
- **Status:** Open
- **Labels:** `enhancement`, `size:XL`
- **Description:**

Implements #70 - Windows installation program packaging

## Features

- PowerShell build script with embedded Python and PyInstaller modes
- Inno Setup integration for professional installer
- Portable version generation
- API key configuration during installation
- Desktop and Start Menu shortcuts

## Build Options

```
./installer/build.ps1                    # Default (embedded Python)
./installer/build.ps1 -PyInstaller       # Self-contained
./installer/build.ps1 -SkipFrontend      # Skip frontend build
./installer/build.ps1 -Clean             # Clean old builds
```

## Output

- dist/MiroFish_Setup_0.1.1.exe - Windows installer
- dist/MiroFish_Portable/ - Portable version

Closes #70

---

### PR #105: fix: security improvements and error handling fixes
- **Author:** @hobostay
- **Branch:** `fix/security-improvements` -> `main`
- **Created:** 2026-03-09
- **Status:** Open
- **Labels:** `documentation`, `size:L`
- **Description:**

## Overview

This PR fixes multiple security issues and code quality problems found in the project.

## Security Fixes

1. **Hardcoded SECRET_KEY** - `backend/app/config.py`
   - Before: Used hardcoded `'mirofish-secret-key'` as default value
   - After: Generates a random secret key and issues a warning if env variable is not set

2. **DEBUG mode defaults to True** - `backend/app/config.py`
   - Before: `DEBUG` defaults to `True`
   - After: `DEBUG` defaults to `False`, more secure for production

3. **CORS config allows all origins** - `backend/app/__init__.py`
   - Before: `CORS(app, resources={r"/api/*": {"origins": "*"}})`
   - After: Configured via `CORS_ALLOWED_ORIGINS` env variable, defaults to localhost only

4. **Error responses leak traceback** - All API files
   - Before: All API error responses included full traceback
   - After: Traceback only returned in DEBUG mode

## Code Quality Improvements

- Added `backend/app/utils/error_handler.py` - Unified error handling utility
- Fixed empty exception handlers in `file_parser.py`, added proper logging
- Updated `.env.example` with new security configuration options

## Files Modified

```
.env.example                       | 11 +++++++
backend/app/__init__.py            | 18 +++++++++--
backend/app/api/graph.py           | 40 ++++++++++--------------
backend/app/api/report.py          | 34 ++++++++++----------
backend/app/api/simulation.py      | 61 ++++++++++++++++++------------------
backend/app/config.py              | 20 +++++++++---
backend/app/utils/__init__.py      |  3 +-
backend/app/utils/error_handler.py | 63 ++++++++++++++++++++++++++++++++++++++
backend/app/utils/file_parser.py   | 13 +++++---
9 files changed, 182 insertions(+), 81 deletions(-)
```

## Checklist

- [x] Code follows project style
- [x] Changes have been tested
- [x] Relevant documentation updated (.env.example)
- [x] Commit messages clearly describe the changes

---

### PR #104: fix: make vite proxy target configurable via environment variable
- **Author:** @nil957
- **Branch:** `fix/remove-hardcoded-api-url` -> `main`
- **Created:** 2026-03-09
- **Status:** Open
- **Labels:** `documentation`, `size:M`
- **Description:**

## Summary

This PR addresses issue #93 - `baseURL` in `frontend/src/api/index.js` should not be hardcoded

## Problem

While `api/index.js` already supports `VITE_API_BASE_URL`, the vite dev server proxy in `vite.config.js` was still hardcoded to `http://localhost:5001`, causing issues when:
- Deploying with Docker using custom port mappings
- Running backend on a remote server
- Using non-standard ports

## Changes

1. **Updated `vite.config.js`** to read `VITE_API_BASE_URL` from environment variables
2. **Added documentation** in `.env.example` for the new configuration option

## Usage

```bash
# In .env file
VITE_API_BASE_URL=http://your-backend-host:5001
```

If not set, it defaults to `http://localhost:5001` for backward compatibility.

Closes #93

---

### PR #103: ci: upgrade GitHub Actions and add ARM64 Docker support
- **Author:** @nil957
- **Branch:** `fix/upgrade-actions-and-arm-support` -> `main`
- **Created:** 2026-03-09
- **Status:** Open
- **Labels:** `enhancement`, `size:XS`
- **Description:**

## Summary

This PR addresses two issues:
- #92 - Upgrade GitHub Actions
- #99 - Docker image has no ARM version

## Changes

1. **Upgraded docker/build-push-action** from v5 to v6
2. **Added multi-platform build support** for both `linux/amd64` and `linux/arm64`
3. **Added GitHub Actions cache** (`cache-from` and `cache-to`) for faster subsequent builds

## Benefits

- Users on ARM-based machines can now use the official Docker image:
  - Apple Silicon Macs (M1/M2/M3/M4)
  - AWS Graviton instances
  - Raspberry Pi
  - Other ARM64 devices
- Faster CI builds with caching enabled

## Testing

The workflow uses QEMU for cross-platform emulation, which is already set up in the existing workflow.

Closes #92
Closes #99

---

### PR #102: fix(ci): add multi-platform Docker build for ARM64 support
- **Author:** @JasonOA888
- **Branch:** `fix/docker-multiplatform` -> `main`
- **Created:** 2026-03-09
- **Status:** Open
- **Labels:** `enhancement`, `size:XS`
- **Description:**

## Problem

ARM64 machines (e.g., Apple Silicon Macs, ARM servers) cannot deploy MiroFish via Docker:

```
no matching manifest for linux/arm64/v8 in the manifest list entries
```

## Solution

Add `platforms: linux/amd64,linux/arm64` to `docker/build-push-action`.

The workflow already has QEMU and Buildx configured, just needed the platforms flag.

## Changes

- Update `.github/workflows/docker-image.yml`
- Add `platforms: linux/amd64,linux/arm64`

Fixes #99

---

### PR #101: feat(utils): add json_utils module for robust LLM JSON parsing
- **Author:** @JasonOA888
- **Branch:** `feat/json-utils-helper` -> `main`
- **Created:** 2026-03-09
- **Status:** Open
- **Labels:** `enhancement`, `size:M`
- **Description:**

## Problem

Some LLM models don't respect `json_object` format and return markdown-wrapped JSON:

```
```json
{"key": "value"}
```
```

This causes `json.loads()` to fail with JSONDecodeError.

Affected models: MiniMax M2.5, GLM-4.7, GLM-5

Related issues: #72, #64, #58, #48

## Solution

Add `json_utils.py` module with:

- `clean_llm_json_response()` - Strip markdown code blocks
- `parse_llm_json()` - Parse with auto-cleanup, optional default value
- `safe_parse_llm_json()` - Non-throwing version returning `(success, result)`

## Usage

```python
from app.utils import parse_llm_json

# Auto-strips ```json ... ``` wrappers
result = parse_llm_json(llm_response)

# With default fallback
result = parse_llm_json(llm_response, default={})

# Safe version (no exceptions)
success, result = safe_parse_llm_json(llm_response)
```

## Changes

- New file: `backend/app/utils/json_utils.py`
- Update: `backend/app/utils/llm_client.py` uses new helper
- Update: `backend/app/utils/__init__.py` exports new functions

---

### PR #100: fix(frontend): use relative baseURL in production, avoid hardcoded localhost
- **Author:** @JasonOA888
- **Branch:** `fix/frontend-baseurl` -> `main`
- **Created:** 2026-03-09
- **Status:** Open
- **Labels:** `enhancement`, `size:S`
- **Description:**

## Problem

Frontend `baseURL` defaults to `http://localhost:5001` in all environments, causing:

1. Users can only access from the machine running MiroFish
2. Docker deployments with non-5001 ports fail
3. Server deployments require frontend rebuild to change URL

## Solution

Make baseURL environment-aware:

- **Production**: Use relative path (empty string) for same-origin deployment
- **Development**: Keep `localhost:5001` default
- **Override**: `VITE_API_BASE_URL` still takes highest priority

```javascript
const getBaseURL = () => {
  if (import.meta.env.VITE_API_BASE_URL) {
    return import.meta.env.VITE_API_BASE_URL
  }
  if (import.meta.env.PROD) {
    return ''  // relative path in production
  }
  return 'http://localhost:5001'  // dev default
}
```

## Testing

- Dev: Works with localhost:5001
- Prod (Docker): Works with any port mapping
- Custom: Works with `VITE_API_BASE_URL` set

Fixes #93, related to #59, #57

---

### PR #87: Upgrade GitHub Actions to latest versions
- **Author:** @salmanmkc
- **Branch:** `upgrade-github-actions-node24-general` -> `main`
- **Created:** 2026-03-08
- **Status:** Open
- **Labels:** `size:S`
- **Description:**

## Summary

Upgrade GitHub Actions to their latest versions for improved features, bug fixes, and security updates.

## Changes

| Action | Old Version(s) | New Version | Release | Files |
|--------|---------------|-------------|---------|-------|
| `docker/build-push-action` | [`v5`](https://github.com/docker/build-push-action/releases/tag/v5) | [`v7`](https://github.com/docker/build-push-action/releases/tag/v7) | [Release](https://github.com/docker/build-push-action/releases/tag/v7) | docker-image.yml |
| `docker/login-action` | [`v3`](https://github.com/docker/login-action/releases/tag/v3) | [`v4`](https://github.com/docker/login-action/releases/tag/v4) | [Release](https://github.com/docker/login-action/releases/tag/v4) | docker-image.yml |
| `docker/metadata-action` | [`v5`](https://github.com/docker/metadata-action/releases/tag/v5) | [`v6`](https://github.com/docker/metadata-action/releases/tag/v6) | [Release](https://github.com/docker/metadata-action/releases/tag/v6) | docker-image.yml |
| `docker/setup-buildx-action` | [`v3`](https://github.com/docker/setup-buildx-action/releases/tag/v3) | [`v4`](https://github.com/docker/setup-buildx-action/releases/tag/v4) | [Release](https://github.com/docker/setup-buildx-action/releases/tag/v4) | docker-image.yml |
| `docker/setup-qemu-action` | [`v3`](https://github.com/docker/setup-qemu-action/releases/tag/v3) | [`v4`](https://github.com/docker/setup-qemu-action/releases/tag/v4) | [Release](https://github.com/docker/setup-qemu-action/releases/tag/v4) | docker-image.yml |

## Why upgrade?

Keeping GitHub Actions up to date ensures:
- **Security**: Latest security patches and fixes
- **Features**: Access to new functionality and improvements
- **Compatibility**: Better support for current GitHub features
- **Performance**: Optimizations and efficiency improvements

### ⚠️ Breaking Changes

- **docker/setup-qemu-action** (v3 → v4): Major version upgrade — review the [release notes](https://github.com/docker/setup-qemu-acti

*... (truncated)*

---

### PR #86: Upgrade GitHub Actions for Node 24 compatibility
- **Author:** @salmanmkc
- **Branch:** `upgrade-github-actions-node24` -> `main`
- **Created:** 2026-03-08
- **Status:** Open
- **Labels:** `size:XS`
- **Description:**

## Summary

Upgrade GitHub Actions to their latest versions to ensure compatibility with Node 24, as Node 20 will reach end-of-life in April 2026.

## Changes

| Action | Old Version(s) | New Version | Release | Files |
|--------|---------------|-------------|---------|-------|
| `actions/checkout` | [`v4`](https://github.com/actions/checkout/releases/tag/v4) | [`v6`](https://github.com/actions/checkout/releases/tag/v6) | [Release](https://github.com/actions/checkout/releases/tag/v6) | docker-image.yml |

## Context

Per [GitHub's announcement](https://github.blog/changelog/2025-09-19-deprecation-of-node-20-on-github-actions-runners/), Node 20 is being deprecated and runners will begin using Node 24 by default starting June 2nd, 2026.

### Why this matters

- **Node 20 EOL**: April 2026
- **Node 24 default**: June 2nd, 2026
- **Action**: Update to latest action versions that support Node 24

### ⚠️ Breaking Changes

- **actions/checkout** (v4 → v6): Major version upgrade — review the [release notes](https://github.com/actions/checkout/releases) for breaking changes

### Security Note

Actions that were previously pinned to commit SHAs remain pinned to SHAs (updated to the latest release SHA) to maintain the security benefits of immutable references.

### Testing

These changes only affect CI/CD workflow configurations and should not impact application functionality. The workflows should be tested by running them on a branch before merging.


---

### PR #82: [Security] Fix CRITICAL vulnerability: CVE-2025-64712
- **Author:** @orbisai0security
- **Branch:** `fix-cve-2025-64712-unstructured` -> `main`
- **Created:** 2026-03-08
- **Status:** Open
- **Labels:** `size:XS`
- **Description:**

## Security Fix

This PR addresses a **CRITICAL** severity vulnerability detected by our security scanner.

### Security Impact Assessment

| Aspect | Rating | Rationale |
|--------|--------|-----------|
| Impact | Critical | In MiroFish's backend, which likely processes unstructured data including MSG files via the Unstructured library, exploitation could allow an attacker to write arbitrary files on the server, potentially leading to remote code execution or full system compromise by overwriting critical system files or injecting malicious code. |
| Likelihood | High | MiroFish appears to be a backend service handling data processing, making it susceptible if it accepts user-uploaded MSG attachments; attackers with access to file upload endpoints could easily craft malicious MSG files using publicly available tools, given the common use of document processing in such repositories. |
| Ease of Fix | Medium | Remediation involves updating the Unstructured library to a patched version as indicated by the provided commit and advisory, requiring dependency updates in uv.lock, moderate testing to ensure compatibility with MiroFish's data processing workflows, and potential refactoring if API changes are involved. |

### Vulnerability Details
- **Rule ID**: `CVE-2025-64712`
- **File**: `backend/uv.lock`
- **Description**: Unstructured has Path Traversal via Malicious MSG Attachment that Allows Arbitrary File Write

### Changes Made
This automated fix addresses the vulnerability by applying security best practices.

### Files Modified
- `backend/requirements.txt`

### Verification
This fix has been automatically verified through:
- ✅ Build verification
- ✅ Scanner re-scan
- ✅ LLM code review

🤖 This PR was automatically generated.


---

### PR #81: feat: add configurable API timeout for slow local LLMs
- **Author:** @JasonOA888
- **Branch:** `fix/issue-58-configurable-timeout` -> `main`
- **Created:** 2026-03-08
- **Status:** Open
- **Labels:** `LLM API`, `size:XS`
- **Description:**

## Summary

Fixes #58 - Allow users to configure API timeout to support slower local LLMs (e.g. Ollama).

## Changes

- Added `VITE_API_TIMEOUT` environment variable support
- Default remains 300000ms (5 minutes)
- Users can increase timeout as needed

## Usage

Add to your `.env` file:
```bash
# Increase timeout for slower local LLMs
VITE_API_TIMEOUT=600000  # 10 minutes
```

## Testing

- Default value of 300000ms works normally
- After configuration, uses the configured value

Fixes #58

---

### PR #74: fix: replace 4 bare excepts with except Exception
- **Author:** @haosenwang1018
- **Branch:** `fix/bare-excepts` -> `main`
- **Created:** 2026-02-25
- **Status:** Open
- **Labels:** `size:XS`
- **Description:**

Bare `except:` → `except Exception:` in 3 backend files (4 sites).

---

### PR #73: fix: Handle string entities/edges in _validate_and_process
- **Author:** @calvinguo721
- **Branch:** `fix-ontology-validation` -> `main`
- **Created:** 2026-02-20
- **Status:** Open
- **Labels:** `LLM API`, `size:S`
- **Description:**

## Problem
When LLM returns malformed JSON where `entity_types` or `edge_types` contain strings instead of dictionaries, the `_validate_and_process` method crashes with:
```
TypeError: 'str' object does not support item assignment
```

## Solution
Added type checking in `_validate_and_process`:
- If entity/edge is a string, wrap it into proper dict format
- Skip invalid (non-dict) entries to prevent crashes
- Collect valid entries into new lists

## Testing
Tested with DeepSeek model via OpenRouter. Successfully generates ontology without crashes.

---

### PR #72: fix: Clean up markdown markers returned by the model
- **Author:** @MoeclubM
- **Branch:** `main` -> `main`
- **Created:** 2026-02-15
- **Status:** Open
- **Labels:** `size:S`
- **Description:**

## Description
Clean up markdown code fence markers that are sometimes returned by LLM models in their responses, which can cause JSON parsing failures.

---

### PR #70: feat: Windows installer packaging
- **Author:** @Jonah-Wu23
- **Branch:** `main` -> `main`
- **Created:** 2026-02-10
- **Status:** Open
- **Labels:** `enhancement`, `size:XXL`
- **Description:**

## Description
Add Windows installer packaging scripts and configuration using Electron Builder / NSIS to create a native Windows installer for the application.

---

### PR #49: feat: Memory graph localization implementation
- **Author:** @Momoyeyu
- **Branch:** `feat/local` -> `main`
- **Created:** 2026-01-22
- **Status:** Open
- **Labels:** *None*
- **Description:**

## Description
Implement local memory graph functionality as an alternative to cloud-based graph services. This allows running the knowledge graph locally without depending on external services like Zep Cloud.

---

### PR #38: feat: Add support for Anthropic SDK (Claude) and multi-provider switching
- **Author:** @SmartisanNaive
- **Branch:** `feat/anthropic-sdk` -> `main`
- **Created:** 2026-01-20
- **Status:** Open
- **Labels:** *None*
- **Description:**

This PR introduces native support for the **Anthropic (Claude)** SDK to MiroFish!

While the project already supports various models via the OpenAI-compatible interface, integrating the official Anthropic SDK allows us to better leverage the capabilities of models like Claude 4 Sonnet. It also ensures compatibility with providers that follow the Anthropic protocol (e.g., Zhipu AI's GLM-4.7 endpoint).

### 🛠️ Key Changes

1.  **Refactored LLMClient (`backend/app/utils/llm_client.py`)**:
    - Introduced an `LLM_PROVIDER` mechanism, allowing seamless switching between `openai` (default) and `anthropic`.
    - **JSON Mode Adaptation**: Since the Claude SDK lacks a native `json_object` mode, I implemented a fallback strategy within the client using System Prompt injection and Markdown parsing. This ensures a consistent `chat_json` experience for the upper layers.

2.  **Configuration Enhancements (`backend/app/config.py` & `.env.example`)**:
    - Added new config options: `ANTHROPIC_API_KEY`, `ANTHROPIC_BASE_URL`, etc.
    - Organized `.env.example` to clearly distinguish between provider configurations.

3.  **Documentation**:
    - Added `backend/LLM_README.md` with detailed instructions on how to configure and switch providers.

### 🧪 How to Test

1.  Update dependencies:
    ```bash
    cd backend
    uv sync  # or uv pip install -r requirements.txt
    ```

2.  Update `.env` to switch to Anthropic:
    ```ini
    LLM_PROVIDER=anthropic
    ANTHROPIC_API_KEY=your_key
    # ANTHROPIC_BASE_URL=... (optional, for custom endpoints)
    ```

3.  Start the backend service and trigger a prediction task to verify that the logs output correctly.

Verified locally using Zhipu GLM-4.7 (Anthropic protocol) - communication works as expected.

Ready for review! Thanks 🚀

---

### PR #15: fix(frontend): handle simulation failed status
- **Author:** @tt-a1i
- **Branch:** `fix/frontend-simulation-error-handling` -> `main`
- **Created:** 2026-01-06
- **Status:** Open
- **Labels:** *None*
- **Description:**

## Summary

- Add check for `runner_status === 'failed'` in `fetchRunStatus()` to properly display error messages when simulation fails
- Previously the UI would stay in "running" state indefinitely when simulation failed

Closes #14

## AI Assistance Disclosure

I used Codex to review the changes, sanity-check the implementation against existing patterns, and help spot potential edge cases.

---


## Merged PRs

### PR #12: Backend terminal color change
- **Author:** @666ghj
- **Branch:** `cursor/backend-terminal-color-change-14fd` -> `main`
- **Created:** 2025-12-30
- **Status:** Merged
- **Labels:** *None*
- **Description:**

Update `dev` script to change backend terminal prefix color from yellow to green.

---
<a href="https://cursor.com/background-agent?bcId=bc-a5b64111-4551-4e51-bef6-3522fd945bdd"><picture><source media="(prefers-color-scheme: dark)" srcset="https://cursor.com/open-in-cursor-dark.svg"><source media="(prefers-color-scheme: light)" srcset="https://cursor.com/open-in-cursor-light.svg"><img alt="Open in Cursor" src="https://cursor.com/open-in-cursor.svg"></picture></a>&nbsp;<a href="https://cursor.com/agents?id=bc-a5b64111-4551-4e51-bef6-3522fd945bdd"><picture><source media="(prefers-color-scheme: dark)" srcset="https://cursor.com/open-in-web-dark.svg"><source media="(prefers-color-scheme: light)" srcset="https://cursor.com/open-in-web-light.svg"><img alt="Open in Web" src="https://cursor.com/open-in-web.svg"></picture></a>



---


## Closed PRs

### PR #224: feat-001-circular-fab-menu
- **Author:** @truthwatcher-ai
- **Branch:** `feat-001-circular-fab-menu` -> `main`
- **Created:** 2026-03-17
- **Status:** Closed
- **Labels:** `enhancement`, `size:XXL`
- **Description:**

## Summary
- Add global reactive store for seed task state (survives modal close/reopen)
- Refactor SeedGeneratorModal to use global store for background task persistence
- Replace simple language toggle FAB with ring segment arc menu (language + research)
- Auto-recover seed files when returning to home page
- Fix FAB styling for dark backgrounds, progress ring circularity, and status normalization

## Test plan
- [ ] Click FAB diamond → ring segments appear (language left, research up)
- [ ] Click language segment → toggles EN/MS
- [ ] Click research segment → opens seed modal
- [ ] Start seed generation → close modal → FAB shows progress ring
- [ ] Reopen modal → progress resumes from where it was
- [ ] Completed seed files auto-populate upload zone

---

### PR #213: Add Federal Register precision tuning
- **Author:** @JustAHobbyDev
- **Branch:** `feat/federal-register-precision-tuning` -> `main`
- **Created:** 2026-03-16
- **Status:** Closed
- **Labels:** `enhancement`, `size:XXL`
- **Description:**

## Summary

- Add deterministic relevance scoring (positive/negative keyword markers) that classifies Federal Register documents as `directly_relevant`, `adjacent`, or `noise`
- Filter noise documents by default before normalization — eliminates irrelevant notices like wildlife/marine mammals/consumer furnaces
- Conditional process-layer enrichment: only assign layers whose markers actually appear in document text (was blanket-stamping all profile layers on every doc)
- Add 4 narrower decomposed profiles: `rare_earths`, `processed_critical_minerals`, `stockpile_and_section232`, `entity_list_export_controls`
- Adjacent docs get weaker relationship expansion to reduce graph inflation
- Persist relevance metadata (score, class, markers, matched layers) on each normalized feed document

**Result**: Live `rare_earths` fetch went from 5 raw -> 1 kept (NdFeB magnets notice with 1 matched layer). Previous `critical_materials` run had 10 docs with all 5 layers stamped on every doc. Structural parse density dropped from 21 entities / 70 relationships / 10 claims to 7 / 4 / 1.

## Test plan

- [x] 37 tests pass (17 new): relevance scoring, noise filtering, conditional enrichment, end-to-end mixed samples
- [x] Policy feed connector tests still pass (3/3)
- [x] Live `rare_earths` profile fetch executed successfully
- [x] Full pipeline (feed -> source bundle -> structural parse) produces cleaner output

@codex review

🤖 Generated with [Claude Code](https://claude.com/claude-code)

---

### PR #212: Add Federal Register precision tuning
- **Author:** @JustAHobbyDev
- **Branch:** `feat/federal-register-precision-tuning` -> `main`
- **Created:** 2026-03-16
- **Status:** Closed
- **Labels:** `enhancement`, `size:XXL`
- **Description:**

## Summary

- Add deterministic relevance scoring (positive/negative keyword markers) that classifies Federal Register documents as `directly_relevant`, `adjacent`, or `noise`
- Filter noise documents by default before normalization — eliminates irrelevant notices like wildlife/marine mammals/consumer furnaces
- Conditional process-layer enrichment: only assign layers whose markers actually appear in document text (was blanket-stamping all profile layers on every doc)
- Add 4 narrower decomposed profiles: `rare_earths`, `processed_critical_minerals`, `stockpile_and_section232`, `entity_list_export_controls`
- Adjacent docs get weaker relationship expansion to reduce graph inflation
- Persist relevance metadata (score, class, markers, matched layers) on each normalized feed document

**Result**: Live `rare_earths` fetch went from 5 raw -> 1 kept (NdFeB magnets notice with 1 matched layer). Previous `critical_materials` run had 10 docs with all 5 layers stamped on every doc. Structural parse density dropped from 21 entities / 70 relationships / 10 claims to 7 / 4 / 1.

## Test plan

- [x] 37 tests pass (17 new): relevance scoring, noise filtering, conditional enrichment, end-to-end mixed samples
- [x] Policy feed connector tests still pass (3/3)
- [x] Live `rare_earths` profile fetch executed successfully
- [x] Full pipeline (feed -> source bundle -> structural parse) produces cleaner output

cc @codex

🤖 Generated with [Claude Code](https://claude.com/claude-code)

---

### PR #211: docs: add Portuguese Brazilian translation of README
- **Author:** @cyberfika
- **Branch:** `claude/translate-readme-pt-br-nkTH1` -> `main`
- **Created:** 2026-03-16
- **Status:** Closed
- **Labels:** `documentation`, `size:L`
- **Description:**

Translate the original Chinese README.md to Portuguese Brazilian (PT-BR), providing complete documentation for Portuguese-speaking users covering:
- Project overview and vision
- Feature demonstrations and workflow
- Quick start guide with source code and Docker deployment options
- Environment setup instructions
- Contact and collaboration information

https://claude.ai/code/session_01NuuVpXrNW9TRDY6TidbC2q

---

### PR #209: fix: resolve 500 error caused by <think> tags and markdown code fences in content field from reasoning models like MiniMax/GLM
- **Author:** @rumble15
- **Branch:** `main` -> `main`
- **Created:** 2026-03-16
- **Status:** Closed
- **Labels:** `size:XXL`
- **Description:**

*No description provided.*

---

### PR #207: docs: English descriptions
- **Author:** @SebGK
- **Branch:** `main` -> `main`
- **Created:** 2026-03-16
- **Status:** Closed
- **Labels:** `size:L`
- **Description:**

## Description
Add English descriptions/translations to the codebase.

---

### PR #201: docs: add Korean README (README-KO.md)
- **Author:** @Kimchikilla
- **Branch:** `add-korean-readme` -> `main`
- **Created:** 2026-03-16
- **Status:** Closed
- **Labels:** `documentation`, `size:L`
- **Description:**

## Summary
- Add a Korean translation of the English README (`README-KO.md`)
- Includes all sections: overview, vision, quick start, workflow, Docker deployment, etc.
- Added Korean language link in the header navigation

## Motivation
To improve accessibility for Korean-speaking contributors and users.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

---

### PR #199: feat(backend): Add Azure OpenAI provider and LLM provider switch
- **Author:** @gonewiththeway
- **Branch:** `feat/azure-openai-llm` -> `main`
- **Created:** 2026-03-15
- **Status:** Closed
- **Labels:** `documentation`, `enhancement`, `LLM API`, `size:L`
- **Description:**

## Summary
Adds support for Azure OpenAI as a second LLM provider alongside the existing OpenAI-compatible API. Backend selects the provider via `LLM_PROVIDER` (`openai` or `azure_openai`).

## Changes
- **Config**: `LLM_PROVIDER`, Azure env vars (`AZURE_OPENAI_API_KEY`, `AZURE_OPENAI_BASE_URL` or `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_DEPLOYMENT`)
- **LLMClient**: Uses Azure config when `provider=azure_openai`; sends `max_completion_tokens` for Azure (replaces `max_tokens` for compatibility)
- **Logging**: Request/response and exception logging in `llm_client` for easier debugging
- **Docs**: `.env.example` and README (zh/en) updated with provider options and env examples. No new Python dependencies; `pyproject.toml` and `uv.lock` unchanged.

Made with [Cursor](https://cursor.com)

---

### PR #194: Translate Chinese text to English and enhance macOS compatibility
- **Author:** @SimonTingle
- **Branch:** `claude/check-macos-compatibility-BoWVb` -> `main`
- **Created:** 2026-03-15
- **Status:** Closed
- **Labels:** `enhancement`, `size:XXL`
- **Description:**

Translate Chinese text to English and enhance macOS compatibility

---

### PR #167: English translation fixes and improvements
- **Author:** @nikay99
- **Branch:** `master` -> `main`
- **Created:** 2026-03-12
- **Status:** Closed
- **Labels:** `documentation`, `size:XXL`
- **Description:**

*No description provided.*

---

### PR #152: feat(report): Zep naming fix and export Markdown functionality
- **Author:** @sx-tane
- **Branch:** `support-pascal-and-snake-case` -> `main`
- **Created:** 2026-03-11
- **Status:** Closed
- **Labels:** `size:XL`
- **Description:**

## Description
Fix Zep naming conventions and add Markdown export functionality for reports.

### Changes
- Support both PascalCase and snake_case naming for Zep compatibility
- Add Markdown export option for simulation reports

---

### PR #147: feat: Russian localization
- **Author:** @notageek88
- **Branch:** `russian-localization` -> `main`
- **Created:** 2026-03-11
- **Status:** Closed
- **Labels:** `documentation`, `size:XXL`
- **Description:**

## 🇷🇺 Russian Localization

This PR adds a complete Russian translation of MiroFish:

### Changes:
- **15 Vue components** — all UI labels, buttons, placeholders, error messages, and tooltips translated from Chinese to Russian
- **README-RU.md** — full Russian documentation with quick start guide
- Translation files are in `frontend-ru/src/` (ready to merge into `frontend/src/` when approved)
- LLM prompts kept in English for compatibility with all providers

### Why:
MiroFish is a brilliant project and we believe Russian-speaking developers and researchers will benefit greatly from native language support. Russia has a strong AI/ML community that would love to use MiroFish for predictive simulations.

### Translation by:
- **Artem Mashin** — Telegram: [@aa_mashin](https://t.me/aa_mashin) | Threads: [@mashin_aa](https://www.threads.com/@mashin_aa)
- **СИНДИКАТ AI** project

---

We're happy to adjust anything based on your feedback. Great project! 🐟

---

### PR #130: docs: Add contribution guide document
- **Author:** @M-Tlinqinming
- **Branch:** `docs/add-pr-guide` -> `main`
- **Created:** 2026-03-11
- **Status:** Closed
- **Labels:** `documentation`, `size:M`
- **Description:**

## Description
Add contribution guide documentation to help new contributors get started with the project.

---

### PR #127: Fix potential crash in LLMClient when content is None
- **Author:** @sjhddh
- **Branch:** `fix/llm-client-none-content` -> `main`
- **Created:** 2026-03-10
- **Status:** Closed
- **Labels:** `LLM API`, `size:XS`
- **Description:**

Added `if content is None: return ""` in `backend/app/utils/llm_client.py` to prevent `re.sub` TypeError.

---
*Automated PR created by OpenClaw daily-pr routine.*

---

### PR #120: fix: Fix neo4j_client import path error in subsystems directory; feat: Add TODO.md development planning document
- **Author:** @28764116
- **Branch:** `main` -> `main`
- **Created:** 2026-03-10
- **Status:** Closed
- **Labels:** `size:XXL`
- **Description:**

## Description
- Fix neo4j_client import path error in the subsystems directory
- Add TODO.md development planning document

---

### PR #97: Feature/local graphrag
- **Author:** @Jerry050512
- **Branch:** `feature/local-graphrag` -> `main`
- **Created:** 2026-03-09
- **Status:** Closed
- **Labels:** `Memory Layer`, `size:XXL`
- **Description:**

implement local GraphRAG to replace Zep dependency

---

### PR #91: Looking for a stable VPN proxy (spam/off-topic)
- **Author:** @leon-x-labs
- **Branch:** `main` -> `main`
- **Created:** 2026-03-08
- **Status:** Closed
- **Labels:** `size:XS`
- **Description:**

(Off-topic: Request for VPN proxy recommendations - not a code change)

---

### PR #89: feat(graph): Update graph rendering with Sigma.js and integrate new data pipeline
- **Author:** @zzfe-501
- **Branch:** `main` -> `main`
- **Created:** 2026-03-08
- **Status:** Closed
- **Labels:** `enhancement`, `size:XL`
- **Description:**

## Description
Update graph rendering to use Sigma.js and integrate new data pipeline for improved visualization.

---

### PR #44: chore: Add Docker and GitHub Actions support
- **Author:** @moonhalf-nostar
- **Branch:** `main` -> `main`
- **Created:** 2026-01-21
- **Status:** Closed
- **Labels:** *None*
- **Description:**

## Description
Add Docker support and GitHub Actions CI/CD pipeline for automated builds and deployment.

---

### PR #33: chore: Add Docker Compose deployment support
- **Author:** @zouyonghe
- **Branch:** `main` -> `main`
- **Created:** 2026-01-17
- **Status:** Closed
- **Labels:** *None*
- **Description:**

## Description
Add Docker Compose deployment support for easier multi-container orchestration.

---

### PR #25: chore: Add Docker support and CI/CD automation
- **Author:** @Deroino
- **Branch:** `main` -> `main`
- **Created:** 2026-01-15
- **Status:** Closed
- **Labels:** *None*
- **Description:**

## Description
Add Docker containerization support and CI/CD automation via GitHub Actions.

### Changes
- Add Dockerfile for backend and frontend
- Add docker-compose.yml for full stack deployment
- Add GitHub Actions workflows for build and test

---

### PR #13: feat: v0.2.0-beta - History UI, resume/cache, native tool calls (development preview)
- **Author:** @martin0359
- **Branch:** `feat/local-resume-history-smoke` -> `main`
- **Created:** 2026-01-03
- **Status:** Closed
- **Labels:** *None*
- **Description:**

## Description
Development preview of v0.2.0-beta featuring:
- History UI for managing past simulation projects
- Resume/cache support to continue interrupted simulations
- Native tool calls integration
- Various UX improvements

---

### PR #10: Codebase bug resolution
- **Author:** @666ghj
- **Branch:** `cursor/codebase-bug-resolution-07f6` -> `main`
- **Created:** 2025-12-29
- **Status:** Closed
- **Labels:** *None*
- **Description:**

Fixes a path traversal vulnerability, a potential infinite loop in text chunking, and a memory leak in the TaskManager to improve security, stability, and performance.

---
<a href="https://cursor.com/background-agent?bcId=bc-d42b0775-c487-403e-a9bf-70f257fe3ce8"><picture><source media="(prefers-color-scheme: dark)" srcset="https://cursor.com/open-in-cursor-dark.svg"><source media="(prefers-color-scheme: light)" srcset="https://cursor.com/open-in-cursor-light.svg"><img alt="Open in Cursor" src="https://cursor.com/open-in-cursor.svg"></picture></a>&nbsp;<a href="https://cursor.com/agents?id=bc-d42b0775-c487-403e-a9bf-70f257fe3ce8"><picture><source media="(prefers-color-scheme: dark)" srcset="https://cursor.com/open-in-web-dark.svg"><source media="(prefers-color-scheme: light)" srcset="https://cursor.com/open-in-web-light.svg"><img alt="Open in Web" src="https://cursor.com/open-in-web.svg"></picture></a>



---

### PR #6: feat(frontend): Add projects history page, default to show only environment-ready projects
- **Author:** @jwc19890114
- **Branch:** `feat/projects-history` -> `main`
- **Created:** 2025-12-26
- **Status:** Closed
- **Labels:** *None*
- **Description:**

## Description
Add projects history page to the frontend. By default, only shows projects where environment setup is complete. Click to enter the environment/project.

---

### PR #2: fix: Encoding display fix
- **Author:** @huanchong-99
- **Branch:** `encoding-display-fix` -> `main`
- **Created:** 2025-12-22
- **Status:** Closed
- **Labels:** *None*
- **Description:**

## Description
Fix encoding display issues in the application.

---

### PR #1: fix: Windows compatibility fix
- **Author:** @huanchong-99
- **Branch:** `bugfix` -> `main`
- **Created:** 2025-12-22
- **Status:** Closed
- **Labels:** *None*
- **Description:**

## Description
Fix Windows compatibility issues.

---

