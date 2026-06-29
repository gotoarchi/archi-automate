# Archi Automate — Capability Reference

A plain-language reference of what Archi Automate can do across hosts. For setup, see [`connect.md`](connect.md); for the product overview, see https://goto.archi/ai-for-aec.

All capabilities are exposed as MCP tools, namespaced per host (`revit_*`, `rhino_*`, `archicad_*`, `ifc_*`), with a cross-host discovery tool so the AI can target any open session.

---

## Across every host
- **Natural-language query** — ask about elements, parameters, quantities, and structure without knowing the host's commands.
- **Analysis** — summarize models, find inconsistencies, check data completeness.
- **Automation** — the AI composes multi-step operations for you; for Revit and Rhino this spans the host's full capability surface, not a fixed command list.
- **Multi-session, multi-host** — one conversation can reach Revit, Rhino, Archicad, and openBIM at the same time and carry intent between them.
- **Voice** — available wherever your connected AI client provides it.

## Revit (2025 / 2026 / 2027)
- Read and query any open model; author and modify elements; governed writes with automatic rollback on failure.
- Fast, recoverable signal when Revit is busy (starting, modal dialog, mid-command) instead of hanging.

## Rhino 8
- Read and query geometry and document data; author and modify; governed writes wrapped in a single undo step.

## Archicad 29
- A curated, governed command set: read model and project data, and create / move / delete elements.

## openBIM / IFC (no CAD app required)
**IFC**
- Open a single model or **federate** several discipline models into one session.
- Read, query, and author the model in memory across IFC2x3 / IFC4 / IFC4x1 / IFC4x3 (schema auto-detected); save to disk.
- Render a **visual snapshot** (PNG the AI can see).
- Export **COBie** (`.xlsx`, FM handover) and **glTF** (`.glb`).

**IDS (buildingSMART Information Delivery Specification)**
- Validate a model against an IDS — pass/fail per requirement.
- Schema / EXPRESS **health check**.
- Search the **buildingSMART Data Dictionary (bSDD)** for authoritative codes.
- **Author new IDS** from a plain-language specification.

**BCF (BIM Collaboration Format)**
- Read and author `.bcfzip` coordination issues (topics, statuses, assignees, comments, referenced elements, embedded snapshot + camera).
- Turn IDS failures or clashes into a coordination issue set — with pictures — in a single step.

**Analysis & sustainability**
- **Clash detection** — geometric, federation-aware.
- **Quantity take-off.**
- **Classification authoring** — Uniclass / OmniClass / bSDD.
- **Embodied-carbon estimation** — EN 15978 / EN 15804 (A1–A3, optional whole-life), with open EPD data.

## Simulation & visualization (cross-host)
- **Scene Contract** — IFC, Revit, and Rhino export the same portable scene, so everything downstream behaves identically regardless of source tool.
- **Local 3D viewer** — the AI launches and drives it (layers, time, colormap, camera, selection) and captures a PNG to "see" the result. Fully local.
- **Solar-access (sun) study** — offline, no network; hours of direct sun per element over a date/time window, computed from the model's own geometry.
- **Site context** — geocode an address and pull surrounding building footprints from public open data into a context massing.
- **AR export** — export to **USDZ** (Apple AR Quick Look) and glTF.

---

## Compatibility
- **AI clients:** any MCP-capable client. One-click for Claude (Desktop & Code), OpenAI Codex, Cursor, VS Code (Copilot agent), Windsurf, Antigravity, Gemini CLI, Cline; guided setup for JetBrains AI Assistant.
- **Recommended models** for reliable AI-composed automation: Claude Opus-class High or better; GPT-5.5 High or better.
- **Platform:** Windows.

See https://goto.archi/ai-for-aec for the full overview and a 14-day free trial.
