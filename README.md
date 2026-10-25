# Archi Automate — AI for AEC

**One AI connection for Revit, Archicad, Rhino, AutoCAD, Vectorworks, SketchUp, Blender, QGIS, Unreal Engine, openBIM/IFC, Tekla Structures through the Dynamic Bridge — and Autodesk Forma and Construction Cloud in the cloud. New in 27.33: start your free trial or enter your license key right in setup.**

Archi Automate is a commercial **MCP (Model Context Protocol) server** that connects AI assistants — the Claude (Anthropic), GPT/Codex (OpenAI), and Gemini (Google) families — to your **live AEC models**. Query, analyze, validate, and automate with natural language, across multiple design tools at once, from the AI client you already use.

> **AI for AEC, not just one CAD tool.** A single MCP connection reaches every supported host, so one conversation can read context in one tool and act in another — including a cloud project with nothing installed at all.

🔗 **Product:** https://goto.archi/ai-for-aec · **User guide:** https://goto.archi/archi-automate-user-guide

[![Website](https://img.shields.io/badge/website-goto.archi-0f62fe)](https://goto.archi/ai-for-aec)
[![MCP](https://img.shields.io/badge/protocol-MCP-5b9bff)](https://modelcontextprotocol.io)
[![Trial](https://img.shields.io/badge/trial-14--day%20free-2e7d32)](https://goto.archi/ai-for-aec)
[![Version](https://img.shields.io/badge/version-27.33-4d4d4d)](https://goto.archi/ai-for-aec)

---

## What's new in 27.33

- **Start your trial in setup.** On a fresh install, setup now has an **Activate** step after the license agreement. On a new computer, choose **Start a free 14-day trial**, enter your name and email, and enter the 6-digit PIN we email you. Or choose **I have a license key**: setup checks the key and saves it.
- **Nothing to register when you are already licensed.** A floating seat, or a key already stored on the computer, needs no registration. A computer whose trial is already running registers it, and the trial keeps its end date. An expired trial does not restart: enter a key now or later in the Hub.
- **No email?** Check your spam folder. **Resend PIN** is available after 30 seconds, and **Change email** lets you correct the address.
- **Never blocked.** If setup cannot reach Archi, choose **Retry** or **Continue without registering**: the install goes on.
- **Updates do not ask again.** Updating from an older version, with setup or the automatic update, skips the Activate step.
- **The full license agreement.** Setup now shows the Archi Automate end-user license agreement (Archi Systems AS): trial, Desktop, Floating and subscription terms, what the license checks send, and your part when an AI assistant works in your models.
- **Fixed:** license checks on computers set to a non-Gregorian calendar (for example Thai Buddhist or Hijri), which could refuse a paid license; pasting into a setup field no longer closes setup. The Rhino plug-in's details now point to https://goto.archi and software@goto.archi.
- No change to the tools in Revit, Archicad or the other applications.

## Earlier releases

**27.32** — Archicad: the AI uses Archicad's own functions, the ones Archicad offers to add-ons: **595 of the 606** useful ones in Archicad 30 (93 to 95% in Archicad 27, 28 and 29), for views and layouts, project preferences and tool defaults, documentation, quantities and data, hotlinks, issues and BCF, Teamwork reservations, libraries, new GDL objects and freeform Morph elements. Changes Archicad cannot undo run in a separate step, and the answer says that Undo does not remove them. Guardrails: deletes are allowed unless "Never delete elements" is on (off by default since 27.32; settings saved before 27.32 keep their value; a company policy can only turn it on). Archicad users run the installer once, then restart Archicad: the automatic update cannot replace the add-on inside Archicad.

**27.31** — Archicad: the AI can create, read and change any element, with every setting: 71 element types and about 2,400 settings in Archicad 30, with geometry, doors, windows and objects with their GDL parameters, and materials, surfaces, layers and composites by name.

**27.30** — Unreal Engine: scenes exported from Revit, IFC, Rhino and Forma now import standing up.

**27.29** — **Archicad 30** support. In Archicad 29 and 30 the AI reads and draws in 2D, lists and opens views, and sends its work as all-or-nothing batches that are one undo step and can be previewed (Archicad 27 and 28 have these since 27.31). A new **Unreal Engine** bridge: the AI reads the open level, runs read-only Python probes and undoable write tasks inside the Unreal Editor, exports the level and imports scenes.

**27.28** — new image generation models (FLUX 3 Image, Nano Banana 2.1, FLUX.2 klein 4B and more through fal.ai), and the **Dynamic Bridge**, which works with Tekla Structures: your AI assistant reads the API of your own installation and writes the code as you ask, on your machine. It replaces the `tekla_*` tools of 27.27 and earlier. Free guide: [Working with Tekla Structures through the Dynamic Bridge](guides/tekla-structures/).

**27.27** finds every installed Archicad 27, 28 and 29 and installs the matching add-on, on install and on repair.

**27.26** added AI image generation from your own model: the viewer's depth, hidden-line and mask passes keep your geometry, with your own provider account, a price before you spend, and a provenance file beside every image.

Before that: **27.24** added the **Grasshopper** graph connector and **Revit 2024**; **27.20** added Archicad 27 and 28 alongside 29, and the **Autodesk Construction Cloud / BIM 360** connector — including moving a whole project between hubs or data regions. **Forma** has been a supported cloud host since 27.16.

Tekla and Tekla Structures are trademarks of Trimble. Unreal and Unreal Engine are trademarks or registered trademarks of Epic Games, Inc. Archi Automate is independent and is not affiliated with or endorsed by Trimble or Epic Games.

---

## What it does

Archi Automate exposes your design tools to any MCP-capable AI client through a single server. Tools are namespaced per host (`revit_*`, `archicad_*`, `rhino_*`, `grasshopper_*`, `autocad_*`, `vectorworks_*`, `sketchup_*`, `blender_*`, `qgis_*`, `unreal_*`, `ifc_*`, `forma_*`, `acc_*`), plus the Dynamic Bridge's `dotnet_*` tools, which are the same for every application, image-generation tools (`render_*`, `provider_*`) and a cross-host discovery tool, so the AI can see and act on every open model at once.

- **Ask questions in plain language** — "How many fire-rated doors are on level 3?", "List elements missing a type mark", "Summarize this IFC's spatial structure."
- **Automate modeling and documentation** — drive repetitive or multi-step tasks the AI composes for you across the host's capabilities.
- **Validate and coordinate openBIM** — check models against standards, detect clashes, and raise coordination issues.
- **See and present models** — render snapshots, drive a local 3D viewer, run a solar study, and export for AR.
- **Work across desktop and cloud** — read a cloud site, bring its context into a desktop tool, and send massing back.
- **Work in visual programming** — read, build, debug and lay out **Grasshopper** definitions themselves, not just the geometry they produce.
- **Render presentation images** — turn the model's own geometry into an AI-rendered visual, with the geometry locked, the price known before you spend, and provenance written beside every result.
- **Move cloud projects** — copy an Autodesk project's folders and files to another hub or data region, Revit cloud models included.

For Revit, Archicad, Rhino, AutoCAD, Vectorworks, SketchUp, Blender, QGIS and Unreal Engine, the AI can compose operations across the host's full capability surface — not just a fixed command list. In Archicad it looks up the element setting or Archicad function it needs before it acts. Through the **Dynamic Bridge** it does the same for Tekla Structures: it looks the API up in your own installation and writes the code. **All write operations are policy-gated, checked before they run, and fully audited** (see [Governance & privacy](#governance--privacy)).

---

## Supported hosts

| Host | Versions | Integration |
|------|----------|-------------|
| **Autodesk Revit** | 2024, 2025, 2026, 2027 | In-process add-in |
| **Autodesk AutoCAD** | 2025, 2026, 2027 | In-process plug-in (per-release bundle) |
| **Graphisoft Archicad** | 27, 28, 29, 30 | Native add-on + bridge (signed, per version) |
| **Rhino (McNeel)** | 8 — incl. **Grasshopper** | Rhino plug-in |
| **Dynamic Bridge** — works with Tekla Structures | Applications with a .NET Framework API, 64-bit; tested with Tekla Structures 2026 | Nothing installed into the application; your assistant sets it up the first time you ask. No read-only mode |
| **Vectorworks** | 2026 | In-process plug-in |
| **Trimble SketchUp** | 2026 | Ruby extension |
| **Blender** | current releases | Python add-on |
| **QGIS** | current releases | Python plug-in |
| **Unreal Engine** | tested with 5.8 | Engine plug-in, editor only, through Epic's Python Editor Script Plugin |
| **openBIM / IFC** | IFC2x3 · IFC4 · IFC4x1 · IFC4x3 | Headless connector — **no CAD app or host license required** |
| **Autodesk Forma** | cloud service | **Cloud connector — nothing to install**; browser sign-in |
| **Autodesk Construction Cloud / BIM 360** | cloud service (US · EMEA) | **Cloud connector — nothing to install**; browser sign-in |

openBIM works on IFC exported from **any** authoring tool (Revit, Archicad, Rhino, Tekla, MicroStation, …), independent of vendor.

Archicad 27, 28, 29 and 30 are each served by their own signed add-on built from the same source; the installer deploys the build that matches every Archicad it finds. The Archicad features of 27.29 to 27.32 were tested live in Archicad 29 and 30; Archicad 27 and 28 run the same add-on code.

Unreal Engine works in the Unreal Editor only and adds nothing to packaged projects. The installer adds the Archi Automate engine plug-in to each Unreal Engine it finds.

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

**Autodesk Construction Cloud / BIM 360 (cloud)**
- Reach **hubs and projects across data regions** — a US and an EMEA hub in the same conversation — walk folder trees, and read item, version, permission and membership detail, with **no local installation**.
- **Download and upload** project files with per-version integrity verification; create folders, items and new versions, or whole **projects** with their initial members.
- **Move a project to another hub or region** — BIM 360 → ACC, US → EMEA — file by file and verified, **including republishing Revit cloud models**, which no Autodesk service does for you.
- Stated plainly: latest versions travel, version history stays with the source project, and issues, markups and permissions do not move.

**Analysis & sustainability** — geometric **clash detection** (federation-aware), **quantity take-off**, classification authoring (**Uniclass / OmniClass / bSDD**), and **embodied-carbon estimation** (EN 15978 / EN 15804, with open EPD data).

**Simulation & visualization (cross-host)**
- A portable **Scene Contract** exported identically from IFC, Revit, Rhino, and Forma.
- A **local 3D viewer** the AI launches, drives, and screenshots — fully local, nothing uploaded.
- An offline **solar-access (sun) study** computed against the model's own geometry.
- **Site context** from public open data (address → surrounding building footprints).
- **AR export** to **USDZ** (Apple AR Quick Look) alongside glTF.

**Archicad (27, 28, 29, 30)**
- **Every element, every setting** (since 27.31) — create, read and change any element type with its documented settings: slabs, roofs, columns, beams, meshes, zones, hatches, curtain walls, stairs and railings with geometry; doors, windows, skylights and objects with their GDL parameters; materials, surfaces, layers and composites by name.
- **Archicad's own functions** (since 27.32) — views and layouts, project settings, documentation, quantities and data, hotlinks, issues and BCF, Teamwork reservations, libraries, new GDL objects and freeform Morph elements.
- **2D, views and batches** (since 27.29) — read and draw in plans, sections, elevations and layouts. Every batch is all or nothing and one undo step, and can be previewed.
- **Reading never changes the model** and works in every Guardrails mode. Changes Archicad cannot undo run in a separate step, and the answer says so.
- **Limits.** A new polygon wall cannot be created through Archicad's API; existing library parts are not edited in place; working in Teamwork projects is not supported yet, although the AI can reserve elements.

**Unreal Engine (since 27.29)**
- **Read the open level** — sub-levels, selection, viewport camera, and actors with their class, transform, bounds, tags, components, meshes and materials; list the project's assets.
- **Python probes and write tasks** inside the Unreal Editor. A write task is one editor transaction: one Ctrl+Z takes it back, and a task that fails part-way is rolled back. A preview runs the task, reports and reverts it.
- **Scenes in and out** — export the level as glTF, and import `.glb`, `.gltf`, `.fbx`, `.obj` and Datasmith files, including the scenes Archi Automate exports from Revit, Rhino, Blender and IFC.
- **Limits.** Editor only; tested with Unreal Engine 5.8; "Never delete elements" does not apply there; imports cannot be previewed or undone; the AI does not save, open or switch levels.

**Dynamic Bridge (since 27.28)**
- **Nothing is pre-programmed.** Archi ships no tools, no code and no settings for any particular application. Your assistant finds the installation, shows you who signed its API files, and stores a small **profile** on your machine: the installation folder, the API files to use and the objects a piece of code starts with.
- **Looks the API up before it writes.** Types, members and the publisher's own documentation text are read from your installed files, so the assistant does not guess names. Then it writes a short piece of C#, runs it, reads the result and goes on, one step at a time.
- **Governed like every bridge.** The code-safety rules (no file, process, network or reflection access by default) apply before code runs; the licence and the audit log apply, and the audit record says which profile the code ran against. A profile is accepted only for an installation under Program Files whose API files carry a valid publisher signature.
- **On your machine.** Everything runs on your computer, with your own installed application and your own licence for it. Archi's servers take no part in running the code (only the Archi Automate licence is checked), and Archi never sees the code, the model or project data.
- **Limits.** No read-only mode; nothing is undone, committed or saved for you; applications with a .NET Framework API, 64-bit; no 3D viewer or rendering export for these applications in this release; a capable AI model is needed.

**Grasshopper (inside Rhino)** — the AI works on the definition itself, not only its output: inspect the live graph, its wires and computed values, search the installed component catalogue, and apply batched edits that land as **one undo step** and roll back entirely if any part fails. It can auto-lay-out a graph left-to-right by dependency depth so the data flow is readable, build and explode clusters, export a picture of the whole definition to look at what it built, and bake results into Rhino as a single, explicit undo step. Diagnostics name the component that **actually** broke rather than the dozens downstream of it. Script-component code sits behind a separate, administrator-enabled capability.

**AI image generation (since 27.26)**
- Render from **your model, not a text prompt alone** — the local viewer's beauty, **depth**, **hidden-line** and **selection-mask** passes become the control input, so the building you designed is the building in the picture.
- Five ways to work: a fresh image from a prompt, a **restyle** that keeps your composition, a **geometry-locked** render driven by depth or line art, **inpainting** of just a masked area, and **upscaling**.
- **Your provider account, your keys** — Google (Gemini), OpenAI, Black Forest Labs (FLUX) and fal.ai; keys are entered in the Hub, encrypted for your Windows user, and never returned to the assistant.
- **Costed before it runs** — a free estimate for any model and size, an optional monthly cap, and a local ledger of what was actually spent.
- **Provenance beside every image** — source image and SHA-256, camera, model, prompt as sent, size, seed, cost and watermark kind, so an AI-assisted visual can be disclosed and audited.
- The model catalogue is **data you can edit**, and a raw passthrough reaches models newer than your build.

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
2. **Activate, in setup** — start your **14-day free trial** (full-featured, all hosts): enter your name and email, then the 6-digit PIN we email you. Or choose **I have a license key**. A floating seat needs no registration, and updates never ask again.
3. **Connect your AI client** — one click from the Archi Automate **Hub**, or paste the standard MCP stdio config. Restart the client.
4. **Ask away** — open a model, point your AI client at it, and start in natural language.

No PIN email? Check your spam folder, wait 30 seconds and click **Resend PIN**, or **Change email**. No connection? **Retry**, or **Continue without registering**: the install is never blocked. Got a key later? Enter it in the Hub under **License** → **Enter license key**; your key is on your account page at goto.archi.

For the cloud connectors — Forma and Construction Cloud / BIM 360 — there is nothing to install: sign in to Autodesk from the Hub and register the project you want to work with. Cloud projects also require a one-time authorization of Archi Automate by an account administrator, once per Autodesk account and region — the Hub shows you exactly what to send them.

Full step-by-step setup, per-client instructions, and troubleshooting are in the [**user guide**](https://goto.archi/archi-automate-user-guide).

---

## Governance & privacy

- **Writes are policy-gated** — read access is open by default; write operations run only where your policy permits, and are **checked before they run** and **fully audited**. A new installation starts in **Allow changes**; choose Read only or Preview changes on the Hub's Guardrails page whenever you want the AI to look without touching.
- **"Never delete elements"** — off by default since 27.32. While it is on, the AI cannot delete elements in Archicad, Grasshopper or Forma. A company policy can turn it on for every managed seat, never off. In the other applications the setting does not apply: use Read only or Preview changes to keep changes out.
- **The Dynamic Bridge has no read-only mode.** Its code runs only in Allow changes; Read only refuses it and Preview changes checks it without running it. Looking the API up works in every mode.
- **A local control center (the Hub)** monitors every host connection and lets administrators edit guardrails graphically.
- **Local by default** — scene export, the 3D viewer, and the solar study run entirely on your machine; **no model data is uploaded**. The only optional outbound calls are public open-data lookups for site context (address geocoding and surrounding footprints) — no model data leaves your machine.
- **Cloud connectors never see your password.** Sign-in happens in your own browser on the vendor's page; Archi Automate receives only a revocable access token, stored encrypted for your Windows user account on your machine. Nothing is held on our servers, and you can revoke access at any time from your vendor account or by disconnecting in the Hub. A cloud connector can only ever reach what the signed-in user is already permitted to see.
- **AI image generation is bounded by the same policy** — it can be switched off entirely, restricted to named providers, capped by monthly budget and image size, and put in **confidential-project mode**, where every request carries the provider's no-store switch and any model whose provider trains on customer inputs is refused outright. Provider keys can be locked to Hub entry so a key never passes through a chat.
- **AI-generated images are labelled** — outputs carry the provider's watermark (Google SynthID, OpenAI C2PA) and a provenance sidecar; disclose AI-assisted imagery where a client or competition requires it.
- **License checks send no project data** — only technical identifiers of the computer and the software's release date. The license agreement in setup says exactly what, and how a trial registration is used.
- **Your AI provider** handles your prompts per its own terms; Archi Automate adds the governed bridge between the AI and your models.

---

## Learn more

- **Product overview:** https://goto.archi/ai-for-aec
- **User guide:** https://goto.archi/archi-automate-user-guide
- **Free guide — Working with Tekla Structures through the Dynamic Bridge:** [`guides/tekla-structures/`](guides/tekla-structures/)
- **MCP for Revit:** https://goto.archi/blog/post/mcp-for-revit
- **MCP for Rhino:** https://goto.archi/blog/post/mcp-for-rhino
- **MCP for Archicad:** https://goto.archi/blog/post/mcp-for-archicad
- **MCP for IFC / openBIM:** https://goto.archi/blog/post/mcp-for-ifc-openbim
- **Connect Claude / Cursor / Codex / VS Code to AEC:** https://goto.archi/blog/post/connect-claude-to-aec

---

## About

Archi Automate is built by **Archi (Archi Systems AS)**, Oslo, Norway — makers of professional automation tools for AEC since 2011.

**Note:** Archi Automate is **commercial software**. This repository is the official **integration and documentation** hub — it does not contain the product source. Questions, setup help, and issue reports are welcome via [Issues](../../issues) or https://goto.archi.

© Archi Systems AS. "Archi" and "Archi Automate" are trademarks of Archi Systems AS. Revit, AutoCAD, Forma, Autodesk Construction Cloud, BIM 360, Archicad, Rhino, Vectorworks, SketchUp, Blender, QGIS, Unreal Engine, Claude, GPT, Gemini, and other names are trademarks of their respective owners. Tekla and Tekla Structures are trademarks of Trimble. Unreal and Unreal Engine are trademarks or registered trademarks of Epic Games, Inc. in the United States of America and elsewhere. Archi Automate is independent and is not affiliated with or endorsed by Trimble or Epic Games.
