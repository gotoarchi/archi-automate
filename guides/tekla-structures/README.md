# Working with Tekla Structures through the Dynamic Bridge

For an AI assistant, and the engineer reading over its shoulder, that writes small C# snippets
against a running Tekla Structures through the Dynamic Bridge of Archi Automate. It is not a program
and has no code to copy; the one longer block, in chapter 0, is an example profile to adapt. It
explains how Tekla Structures is organised, which class is the way in for each job, the order
things must happen in, what goes wrong without an error, and what can cost the user their work.

## What the Dynamic Bridge gives you

The Dynamic Bridge knows nothing about Tekla Structures and has no tools made for it. You look the
API up in the user's own installation and write the code. This guide does not repeat what the tools
say about themselves.

| Tool | What it is for |
|---|---|
| `dotnet_scan_installation`, `dotnet_save_profile`, `dotnet_list_profiles`, `dotnet_delete_profile` | Finding the installation and keeping a **profile**: the folder, the API files, and the usings, aliases and objects every snippet starts with (chapter 0). |
| `dotnet_ping`, `dotnet_list_sessions`, `dotnet_select_session` | Checking that the Dynamic Bridge is up for a profile. |
| `dotnet_api_search` | Type names, member signatures and the publisher's documentation, read from the installed files. It runs nothing. |
| `dotnet_run_code` | The one tool that runs code. It compiles and runs a snippet body with the profile's objects and `result` (a dictionary you fill) in scope. |

Five things follow from that, and every chapter relies on them:

- **What a snippet starts with comes from the profile.** This guide writes `model` for a connected
  `Tekla.Structures.Model.Model` and, in places, `TSD` for the drawing namespace. Those names exist
  only if your profile defines them: `dotnet_list_profiles` shows what yours defines, chapter 0
  shows how to define them. A full type name always works.
- **There is no read-only mode.** The Dynamic Bridge cannot tell a query from a change: whatever the
  snippet calls, happens. A snippet only reads if you put only reads in it.
- **The user's Guardrails setting decides whether code runs.** Read only and Approve changes refuse
  `dotnet_run_code` (looking the API up still works). Preview changes compiles and checks the
  snippet and does not run it. Allow changes runs it.
- **Nothing is undone, committed or saved for you.** A snippet that changes the model calls the
  model's commit itself, once, at the end. A snippet that only reads does not call it.
- **An engineer checks the result.** A snippet that ran without an error is not a checked result.

## How to work

1. **Load only the chapter you need.** Each starts with "Use this chapter when ..." and stands
   alone. Read chapter 8 before the first snippet that changes anything, prints, exports, selects,
   or opens a drawing.
2. **Look up before you guess.** This guide names types and members; `dotnet_api_search` gives the
   exact signature for the installed version.
3. **Read first, small.** Start with a snippet that only reads: it counts and identifies what you
   are about to work on. Return GUIDs and plain values, never live Tekla Structures objects.
4. **For a change:** tell the user what will change, then do it in one run that ends with one
   commit, so it is one undo step.
5. **Check the answer against the model.** A `true` from the API is a claim. Evidence is the object
   read back by its GUID, the file on disk, the selection read back, the count the application
   reported.
6. **Put back** what you changed on the user's screen, if it was yours to change (chapter 8).
7. **Say what you could not read.** "Not in the model", "in the model but unreadable" and "read, and
   empty" are three different answers. Report gaps and anything left in an unexpected state.

## Four facts that hold everywhere

- **Lengths are millimetres**, angles are degrees, in the model and on paper alike.
- **Objects are identified by GUID.** The number the application also hands out is valid for this
  session only.
- **Nothing is committed for you.** A change to the model is one undo step for the user only after
  the snippet that made it has called the commit (chapters 1, 3 and 8).
- **Nothing is saved for you.** Changes live in the open model until the user saves. Never save,
  open or close a model or a drawing on your own initiative.

## How sure each statement is

| Mark | Meaning |
|---|---|
| no mark | Taken from code that runs against Tekla Structures 2026. |
| **[verified 2026]** | Measured or tested live on Tekla Structures 2026. |
| **[lookup only]** | The member exists in the installed API and its documentation says so. Not run by us. |
| **[inferred]** | Our reasoning. Test it with a snippet that only reads before relying on it. |

In another version, trust the lookup and a small test over this guide.

## Chapters

| File | Use it when |
|---|---|
| `00-connecting.md` | You are setting the Dynamic Bridge up for Tekla Structures, or the connection fails. |
| `01-how-tekla-is-organised.md` | You need the mental model: objects, ids, enumerators, commit, units, coordinates, cost. |
| `02-reading-the-model.md` | You read parts, bolts, welds, connections, grids, pours, loads or reinforcement. |
| `03-changing-the-model.md` | You create, modify or delete model objects. |
| `04-drawings.md` | You list drawings, read what is on one, or create and edit drawings. |
| `05-printing-and-exporting.md` | You print PDFs, export IFC or create reports. |
| `06-catalogs.md` | You need a section profile, material, bolt, rebar, shape or component that exists. |
| `07-selection-numbering-clash.md` | You read or set the selection, check numbering, or run the clash check. |
| `08-safety.md` | Before anything that changes the model, a drawing, files or the user's screen. |
| `09-traps.md` | Something returned `true`, `null`, zero or nothing and you do not know why. |

Tekla and Tekla Structures are trademarks of Trimble. This guide is independent and is not affiliated with or endorsed by Trimble.
