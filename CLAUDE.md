# Destined - Critical Context for Claude

## Project Overview

Destined is an agentic product for sales deal management. It builds AI agents around customer profiles and sales-agent profiles, accepts deal context (stage, history, participants, metrics), and recommends next steps that help sales teams move deals toward closed-won.

The repo is currently empty apart from a Python 3.14 virtualenv (`.venv`). No application code exists yet. Do not assume a structure beyond what is documented here until it is built.

## Architecture Goals

**Agent types** (planned, not yet implemented):
- `CustomerAgent` - models the buyer: role, priorities, objections, buying committee position
- `SalesAgent` - models the seller: playbook, strengths, style, quota context
- `DealAnalyzer` - evaluates deal health from stage, history, participants, and metrics
- `NextStepRecommender` - turns analysis into concrete, prioritized actions

**Data models** (planned):
- `Deal` - id, stage, amount, close date, owner, status, stage history
- `Profile` - customer or sales-agent profile (role, company, goals, preferences, history)
- `Interaction` - calls, emails, meetings, notes; timestamp, participants, summary, sentiment
- `Participant` - contacts on a deal, their role in the buying committee, engagement level
- `Recommendation` - action, rationale, expected impact, status (proposed / accepted / dismissed)

**Context flow:**
1. Deal context is assembled from the data layer (Deal + Participants + Interactions + Profiles).
2. `DealAnalyzer` produces a structured assessment (risks, gaps, momentum).
3. Agents receive only the context they need, passed explicitly, not via global state.
4. `NextStepRecommender` returns structured `Recommendation` objects, each with a rationale traceable to specific interactions or metrics.
5. Recommendations and outcomes are persisted so that the feedback loop can measure what worked.

**Design principles:**
- Recommendations must be explainable: every one cites the evidence it is based on.
- Agents are stateless between calls; state lives in the data layer.
- Model and provider calls sit behind a thin interface so they can be swapped and tested without network access.

## Key Directories

Planned layout (create only when the feature needing it is built):

- `src/agents/` - agent implementations (`CustomerAgent`, `SalesAgent`, `DealAnalyzer`, `NextStepRecommender`)
- `src/data/` - data models (`Deal`, `Profile`, `Interaction`, `Participant`, `Recommendation`) and persistence
- `src/context/` - assembling deal context from the data layer for agents
- `src/api/` - API endpoints
- `src/integrations/` - CRM, email, calendar, and LLM provider adapters
- `tests/` - mirrors `src/` structure
- `docs/` - design notes and decisions
- `.venv/` - local virtualenv (Python 3.14); never commit

## Important Rules

**Code quality:**
- Match the style of surrounding code once it exists; keep comments sparse and useful.
- Type-hint all public functions and data models.
- Keep data models as plain, validated structures (e.g. dataclasses or pydantic); no business logic inside them.
- No hardcoded deal stages, thresholds, or playbook content in agent code; put them in configuration.
- Prefer small, single-purpose modules over large orchestrator classes.

**Testing:**
- Every agent and analyzer gets unit tests with fixture deals; tests must not call live LLM or CRM APIs.
- Use recorded or stubbed model responses for any test that exercises prompt-to-output logic.
- Recommendation logic gets golden-file tests: fixed deal inputs produce expected recommendation categories.
- Run the full test suite before reporting a change as done.

**Data and privacy:**
- Deal and customer data is sensitive. Never log full interaction bodies or personal contact details at INFO level.
- Never commit real customer data, API keys, or `.env` files. Use fixtures with synthetic names.
- Every external fetch (CRM, email, enrichment) should persist what it retrieved immediately, so rate-limited or paid calls are not repeated.

**Scope:**
- Do not implement features beyond what the current request asks for.
- Ask before adding new external integrations or paid services.

## Integration Points

Planned, none implemented yet:
- **CRM** (e.g. Salesforce, HubSpot) - source of truth for deals, stages, and contacts
- **Email and calendar** (e.g. Gmail, Google Calendar, Outlook) - interaction history
- **LLM provider** - reasoning for analysis and recommendations, behind an adapter interface
- **Enrichment** (company and contact data) - optional, to be chosen later
- **Notifications** (Slack or email) - delivering recommendations to reps

Each integration must be behind its own module under `src/integrations/` so that the core agents depend on interfaces, not vendors.

## Success Metrics

How we will judge whether Destined is working:
- **Win rate** on deals with Destined recommendations vs. a control group without them
- **Stage velocity** - median days per stage, before vs. after adoption
- **Recommendation acceptance rate** - share of recommendations accepted or acted on by reps
- **Recommendation outcome rate** - share of accepted recommendations followed by a stage advance or a positive customer response within a set window
- **Stalled-deal recovery** - share of deals flagged as stalled that progress within 14 days of a recommendation
- **Forecast accuracy** - gap between forecast close rate and actual close rate

Metrics definitions and measurement windows should be written down in `docs/` before instrumentation is built, so results stay comparable.
