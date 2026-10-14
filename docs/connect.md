# Connecting your AI client

Archi Automate is a standard **MCP stdio server**, so it works with any MCP-capable AI client. There are two ways to connect.

For complete, per-client step-by-step instructions and troubleshooting, see the official **[user guide](https://goto.archi/archi-automate-user-guide)** — this page is a short orientation.

---

## Option 1 — one click (recommended)

After installing Archi Automate, open the **Hub** (its control center) and go to the **LLMs** page. It offers one-click connection for:

- Claude (Desktop & Code)
- OpenAI Codex
- Cursor
- VS Code (Copilot agent)
- Windsurf
- Antigravity
- Gemini CLI
- Cline

The Hub writes the correct configuration for the client and backs up the previous one. Restart the AI client after connecting.

JetBrains AI Assistant uses a short guided manual setup (also in the Hub and the user guide).

---

## Option 2 — manual MCP config

Any MCP client that accepts a stdio server can connect with a standard configuration of this shape:

```jsonc
{
  "mcpServers": {
    "archi-automate": {
      "command": "<path to the Archi Automate MCP server>",
      "args": []
    }
  }
}
```

The installer places the MCP server on your machine and the **Hub shows you the exact command to paste** — use the Hub's copy button rather than hand-typing the path. Restart your AI client after adding the config.

---

## After connecting

1. Open a model in a supported host (Revit, AutoCAD, Archicad, Rhino, Vectorworks, SketchUp, Blender, QGIS), or have an `.ifc` file ready for the openBIM connector. For Tekla Structures, open your model, set Guardrails to Allow changes, and ask your assistant to "connect to my Tekla Structures": the Dynamic Bridge sets itself up. For the cloud connectors — **Autodesk Forma** and **Construction Cloud / BIM 360** — there is nothing to open locally: sign in to Autodesk from the Hub and register your project.
2. In your AI client, ask the assistant to list available sessions or describe the open model.
3. Start with read-only questions; enable write operations through your policy when you're ready (writes are governed and audited).

A **14-day full-featured trial** starts automatically on first run — no license key required, all hosts included.

Questions? Open an [issue](../../issues) or visit https://goto.archi/ai-for-aec.

Tekla and Tekla Structures are trademarks of Trimble. Archi Automate is independent and is not affiliated with or endorsed by Trimble.
