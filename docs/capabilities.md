# Archi Automate — Capability Reference

A plain-language reference of what Archi Automate can do across hosts. For setup, see [`connect.md`](connect.md); for the product overview, see https://goto.archi/ai-for-aec.

All capabilities are exposed as MCP tools, namespaced per host (`revit_*`, `archicad_*`, `rhino_*`, `autocad_*`, `tekla_*`, `vectorworks_*`, `sketchup_*`, `blender_*`, `qgis_*`, `ifc_*`, `forma_*`), with a cross-host discovery tool so the AI can target any open session.

---

## Across every host
- **Natural-language query** — ask about elements, parameters, quantities, and structure without knowing the host's commands.
- **Analysis** — summarize models, find inconsistencies, check data completeness.
- **Automation** — the AI composes multi-step operations for you; for most hosts this spans the host's full capability surface, not a fixed command list.
- **Multi-session, multi-host** — one conversation can reach every supported tool at the same time and carry intent between them.
- **Voice** — available wherever your connected AI client provides it.

## Revit (2025 / 2026 / 2027)
- Read and query any open model; author and modify elements; governed writes with automatic rollback on failure.
- Fast, recoverable signal when Revit is busy (starting, modal dialog, mid-command) instead of hanging.

## AutoCAD (2025 / 2026 / 2027)
- Read and query drawings — entities, properties, layers, and document structure — across multiple open drawings.
- Governed automation of drawing operations; each supported release is served by its matching plug-in build.

## Archicad 29
- A curated, governed command set: read model and project data, and create / move / delete elements.

## Rhino 8
- Read and query geometry and document data; author and modify; governed writes wrapped in a single undo step.

## Tekla Structures 2026
- Read and query the structural model — parts, assemblies, and properties — from a resident bridge that starts on demand.
- Governed automation against the Tekla API.

## Vectorworks 2026
- Read and query document data; create and delete elements; export the scene for downstream simulation.
- Governed automation, with reads and writes dispatched so long queries never block interactive work.

## SketchUp 2026
- Read and query the model and its entities; export the scene.
- Governed Ruby automation inside the running SketchUp session.

## Blender
- Read and query scene data; export the scene.
- **Asset workflow** — search and import assets into the current file.
- Governed Python automation inside the running Blender session.

## QGIS
- Read and query project layers and features.
- **Site context export** — turn geospatial data into context for design and simulation.
- Governed Python automation inside the running QGIS session.

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

## Autodesk Forma (cloud — nothing to install)
The first **cloud host**: there is no add-in, no local application and no host licence on your machine. You sign in to Autodesk from your own browser and register the Forma project you want to work with.

**Read**
- List a project's **proposals** (design alternatives) and read the full element hierarchy.
- Pull the complete site context Forma has already assembled — **terrain, surrounding buildings, roads, property boundaries** — including its geo-reference.
- Export the whole site to the portable **Scene Contract**, so real, geolocated context flows into your desktop tools and the simulation stack.

**Author**
- Create a new **proposal** as a design alternative.
- Contribute **massing geometry** — from a footprint polygon plus a height, or from an existing `.glb` produced anywhere — and place it in the proposal so it is visible to the whole team.
- Placement is **terrain-aware**: give a plan position and the volume is set on the real ground level rather than at elevation zero.
- Describe buildings by **floors** as well as height.
- Remove geometry that Archi Automate contributed.

**Analyse**
- Trigger **Forma's own sun analysis** for the buildings in a proposal and read the result. Site coordinates and the analysed buildings are derived from the project itself.

**Access** — a Forma/Autodesk account administrator authorizes Archi Automate for the account once; after that, each user simply signs in. A cloud connector can only reach what that signed-in user is already permitted to see.

## Simulation & visualization (cross-host)
- **Scene Contract** — IFC, Revit, Rhino and Forma export the same portable scene, so everything downstream behaves identically regardless of source tool.
- **Local 3D viewer** — the AI launches and drives it (layers, time, colormap, camera, selection) and captures a PNG to "see" the result. Fully local.
- **Solar-access (sun) study** — offline, no network; hours of direct sun per element over a date/time window, computed from the model's own geometry.
- **Site context** — geocode an address and pull surrounding building footprints from public open data into a context massing.
- **AR export** — export to **USDZ** (Apple AR Quick Look) and glTF.

---

## Compatibility
- **AI clients:** any MCP-capable client. One-click for Claude (Desktop & Code), OpenAI Codex, Cursor, VS Code (Copilot agent), Windsurf, Antigravity, Gemini CLI, Cline; guided setup for JetBrains AI Assistant.
- **Recommended models** for reliable AI-composed automation: Claude Opus-class High or better; GPT-5.5 High or better.
- **Platform:** Windows. The Forma cloud connector needs only an internet connection and an Autodesk account.

See https://goto.archi/ai-for-aec for the full overview and a 14-day free trial.
