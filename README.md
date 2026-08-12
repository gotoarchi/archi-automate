# Archi Automate — AI for AEC

**One AI connection for Revit, Archicad, Rhino, AutoCAD, Tekla, Vectorworks, SketchUp, Blender, QGIS, openBIM/IFC — and Autodesk Forma in the cloud.**

Archi Automate is a commercial **MCP (Model Context Protocol) server** that connects AI assistants — the Claude (Anthropic), GPT/Codex (OpenAI), and Gemini (Google) families — to your **live AEC models**. Query, analyze, validate, and automate with natural language, across multiple design tools at once, from the AI client you already use.

> **AI for AEC, not just one CAD tool.** A single MCP connection reaches every supported host, so one conversation can read context in one tool and act in another — including a cloud project with nothing installed at all.

🔗 **Product:** https://goto.archi/ai-for-aec · **User guide:** https://goto.archi/archi-automate-user-guide

[![Website](https://img.shields.io/badge/website-goto.archi-0f62fe)](https://goto.archi/ai-for-aec)
[![MCP](https://img.shields.io/badge/protocol-MCP-5b9bff)](https://modelcontextprotocol.io)
[![Trial](https://img.shields.io/badge/trial-14--day%20free-2e7d32)](https://goto.archi/ai-for-aec)

---

## What it does

Archi Automate exposes your design tools to any MCP-capable AI client through a single server. Tools are namespaced per host (`revit_*`, `archicad_*`, `rhino_*`, `autocad_*`, `tekla_*`, `vectorworks_*`, `sketchup_*`, `blender_*`, `qgis_*`, `ifc_*`, `forma_*`), plus a cross-host discovery tool, so the AI can see and act on every open model at once.

- **Ask questions in plain language** — "How many fire-rated doors are on level 3?", "List elements missing a type mark", "Summarize this IFC's spatial structure."
- **Automate modeling and documentation** — drive repetitive or multi-step tasks the AI composes for you across the host's capabilities.
- **Validate and coordinate openBIM** — check models against standards, detect clashes, and raise coordination issues.
- **See and present models** — render snapshots, drive a local 3D viewer, run a solar study, and export for AR.
- **Work across desktop and cloud** — read a cloud site, bring its context into a desktop tool, and send massing back.

For Revit, Rhino, AutoCAD, Tekla, Vectorworks, SketchUp, Blender, and QGIS, the AI can compose operations across the host's full capability surface — not just a fixed command list. **All write operations are policy-gated, checked before they run, and fully audited** (see [Governance & privacy](#governance--privacy)).

---

## Supported hosts

| Host | Versions | Integration |
|------|----------|-------------|
| **Autodesk Revit** | 2025, 2026, 2027 | In-process add-in |
| **Autodesk AutoCAD** | 2025, 2026, 2027 | In-process plug-in (per-release bundle) |
| **Graphisoft Archicad** | 29 | Native add-on + bridge |
| **Rhino (McNeel)** | 8 | Rhino plug-in |
| **Trimble Tekla Structures** | 2026 | Resident bridge |
| **Vectorworks** | 2026 | In-process plug-in |
| **Trimble SketchUp** | 2026 | Ruby extension |
| **Blender** | current releases | Python add-on |
| **QGIS** | current releases | Python plug-in |
| **openBIM / IFC** | IFC2x3 · IFC4 · IFC4x1 · IFC4x3 | Headless connector — **no CAD app or host license required** |
| **Autodesk Forma** | cloud service | **Cloud connector — nothing to install**; browser sign-in |

openBIM works on IFC exported from **any** authoring tool (Revit, Archicad, Rhino, Tekla, MicroStation, …), independent of vendor.

---

## Capabilities

**Model intelligence (all hosts)** — natural-language query, analysis, and automation against live models.

**Desktop hosts** — read and query models, parameters, and document structure; author and modify where your policy allows, with governed writes (rollback on failure in Revit, single-undo in Rhino). Geospatial context comes from QGIS; asset libraries and rendering-oriented work from Blender.

**openBIM — the full standards trio**
- **IFC** — open or **federate** models, read/query/author, save; render a **visual snapshot** the AI can see; export **COBie** (FM handover) and **glTF**.
- **IDS** — validate a model against a buildingSMART **Information Delivery Specification** (pass/fail per requirement), run a schema health check, search the buildingSMART Data Dictionary (**bSDD**), and **author new IDS from plain language**.
- **BCF** — read and author **BIM Collaboration Format** issues (with embedded snapshots + camera); turn IDS failures or clashes into a coordination issue set in one step.

**Autodesk Forma (cloud)**
- Read a Forma project's **proposals**, elements, and full site context — terrain, surrounding buildings, roads, property boundaries — with **no local installation**.
- Export the whole site to the portable **Scene Contract**, so a real, geolocated context flows into your desktop tools and our simulation stack.
- **Author back into Forma:** create design proposals, contribute massing geometry (from a footprint + height, or an existing `.glb`), and place it **automatically grounded on the terrain**.
- Trigger **Forma's own sun analysis** on the buildings in a proposal and read the result.

**Analysis & sustainability** — geometric **clash detection** (federation-aware), **quantity take-off**, classification authoring (**Uniclass / OmniClass / bSDD**), and **embodied-carbon estimation** (EN 15978 / EN 15804, with open EPD data).

**Simulation & visualization (cross-host)**
- A portable **Scene Contract** exported identically from IFC, Revit, Rhino, and Forma.
- A **local 3D viewer** the AI launches, drives, and screenshots — fully local, nothing uploaded.
- An offline **solar-access (sun) study** computed against the model's own geometry.
- **Site context** from public open data (address → surrounding building footprints).
- **AR export** to **USDZ** (Apple AR Quick Look) alongside glTF.

📖 Full capability reference: [`docs/capabilities.md`](docs/capabilities.md)

---

## Works with your AI client

Archi Automate is a standard **MCP stdio server**, so it works with **any MCP-capable client**. One-click connection is built in for:

**Claude (Desktop & Code) · OpenAI Codex · Cursor · VS Code (Copilot agent) · Windsurf · Antigravity · Gemini CLI · Cline** — plus guided setup for **JetBrains AI Assistant**.

Any other current or future MCP client connects with the same copy-paste stdio configuration.

➡️ How to connect: [`docs/connect.md`](docs/connect.md) · [User guide](https://goto.archi/archi-automate-user-guide)

---

## Quick start

1. **Install** — one installer sets up the shared components and every host bridge for the design tools on your machine. Restart any open host (Revit/Archicad/…) after install.
2. **Connect your AI client** — one click from the Archi Automate **Hub**, or paste the standard MCP stdio config. Restart the client.
3. **Start a 14-day free trial** — full-featured, no license key required, covering all hosts.
4. **Ask away** — open a model, point your AI client at it, and start in natural language.

For the cloud connector, there is nothing to install: sign in to Autodesk from the Hub and register the project you want to work with. Cloud projects also require a one-time authorization of Archi Automate by an account administrator — the Hub shows you exactly what to send them.

Full step-by-step setup, per-client instructions, and troubleshooting are in the [**user guide**](https://goto.archi/archi-automate-user-guide).

---

## Governance & privacy

- **Writes are policy-gated** — read access is open by default; write operations run only where your policy permits, and are **checked before they run** and **fully audited**.
- **A local control center (the Hub)** monitors every host connection and lets administrators edit guardrails graphically.
- **Local by default** — scene export, the 3D viewer, and the solar study run entirely on your machine; **no model data is uploaded**. The only optional outbound calls are public open-data lookups for site context (address geocoding and surrounding footprints) — no model data leaves your machine.
- **Cloud connectors never see your password.** Sign-in happens in your own browser on the vendor's page; Archi Automate receives only a revocable access token, stored encrypted for your Windows user account on your machine. Nothing is held on our servers, and you can revoke access at any time from your vendor account or by disconnecting in the Hub. A cloud connector can only ever reach what the signed-in user is already permitted to see.
- **Your AI provider** handles your prompts per its own terms; Archi Automate adds the governed bridge between the AI and your models.

---

## Learn more

- **Product overview:** https://goto.archi/ai-for-aec
- **User guide:** https://goto.archi/archi-automate-user-guide
- **MCP for Revit:** https://goto.archi/blog/post/mcp-for-revit
- **MCP for Rhino:** https://goto.archi/blog/post/mcp-for-rhino
- **MCP for Archicad:** https://goto.archi/blog/post/mcp-for-archicad
- **MCP for IFC / openBIM:** https://goto.archi/blog/post/mcp-for-ifc-openbim
- **Connect Claude / Cursor / Codex / VS Code to AEC:** https://goto.archi/blog/post/connect-claude-to-aec

---

## About

Archi Automate is built by **Archi (Archi Systems AS)**, Oslo, Norway — makers of professional automation tools for AEC since 2011.

**Note:** Archi Automate is **commercial software**. This repository is the official **integration and documentation** hub — it does not contain the product source. Questions, setup help, and issue reports are welcome via [Issues](../../issues) or https://goto.archi.

© Archi Systems AS. "Archi" and "Archi Automate" are trademarks of Archi Systems AS. Revit, AutoCAD, Forma, Archicad, Rhino, Tekla, Vectorworks, SketchUp, Blender, QGIS, Claude, GPT, Gemini, and other names are trademarks of their respective owners.
