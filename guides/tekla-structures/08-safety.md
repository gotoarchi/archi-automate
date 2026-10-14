# 8. Safety

Use this chapter before any snippet that changes the model, a drawing or a catalog, writes a file,
prints, selects objects, opens a drawing, or touches the work plane. Read it once in full.

Essentials: every write takes effect when called and cannot be rolled back by code; nothing is
committed for you, so a snippet that changed the model commits once, at the end, and no other
snippet commits; nothing is saved for you and you must not save; the screen (selection, open
drawing, work plane) is the user's.

You work inside a live session of someone's project, with unsaved work in the model and maybe in an
open drawing. A snippet is safe when the user has lost nothing and is surprised by nothing.

`model` in this chapter is a connected `Tekla.Structures.Model.Model`, if your Dynamic Bridge
profile defines an object under that name; `dotnet_list_profiles` shows what yours defines
(chapter 0).

## What the Dynamic Bridge does not do for you

- **It has no read-only mode.** One tool, `dotnet_run_code`, runs whatever the snippet calls. It
  knows nothing about Tekla Structures and cannot tell a query from a change. The tables below are
  the only guard: know what each call does before you run it, and look it up when unsure.
- **The user's Guardrails setting decides whether code runs at all.** Read only and Approve changes
  refuse `dotnet_run_code`; looking the API up still works. Allow changes runs the snippet. When a
  run is refused, say so and say what the snippet would do: the setting is the user's to change.
- **Preview changes is a compile check, not a preview of the change.** The snippet is compiled and
  checked and not run. Nothing in the answer says what it would have changed.
- **Nothing is undone, committed or saved for you.** What the API needs for a change to count, the
  snippet calls itself: see "No rollback, and one undo step" below.
- **Its code-safety rules** refuse file, process, network and reflection access in a snippet by
  default; the answer is `not_allowed` and names the rule. Do not work around a refusal.
- **Say what a snippet will change before you run one that changes something.** And an engineer
  checks the outcome in the application: a snippet that ran without an error is not a checked
  result.

## What a call can change

| It changes | Examples | Undone by |
|---|---|---|
| **The model** | `Insert`, `Modify`, `Delete` on model objects; `SetUserProperty`; `SetPhase`; `Operation.MoveObject`, `CopyObject`, `Split`, `Combine`; `ProjectInfo.Modify` | The user's undo, one step per commit. Not by code. |
| **A drawing** | `Insert`/`Modify`/`Delete` on drawings and drawing objects; `IssueDrawing`, `UnissueDrawing`, `UpdateDrawing`; `drawing.CommitChanges` | Not by the model's undo. |
| **A catalog** | `Insert`/`Modify`/`Delete`/`Copy` on catalog items; `Import...`; `SaveProfileDatabase` | Nothing. |
| **Files** | `PrintDrawing`, the IFC exports, `CreateReport...`, NC and other exports, a catalog item's `Export`, `Filter.CreateFile` | Nothing. |
| **Only the user's screen** | UI `Select`; `SetActiveDrawing`, `CloseActiveDrawing`; `SetCurrentTransformationPlane`; the calls on `Tekla.Structures.Model.UI.ViewHandler` | You, by putting it back. |
| **Settings, process-wide** | `TeklaStructuresSettings.SetOptions`; unit settings in `Tekla.Structures.Datatype` | Nothing. |
| **The session itself** | `ModelHandler.Save`, `Open`, `Close`, `CreateNew...`; `Operation.Open`; `SaveActiveDrawing`; `Operation.RunMacro` | Nothing. |

Constructors and property setters on API objects are harmless: they touch your in-memory copy until
an `Insert` or `Modify` sends it. Most reads start with `Get`, `Is`, `Has`, `Can`, `Select`,
`Fetch`, `Find`, `Compare` or `Contains`; treat everything else as a write until the lookup says
otherwise. The names that mislead:

| Looks like a read, but writes | Looks like a write, but only reads or computes |
|---|---|
| `DrawingHandler.IssueDrawing` (it starts with "Is") | `Select()` on a model object, drawing object or catalog item: it loads |
| `ProfileItem.GetProfileItemSubTypes`: updates the profile database | `Solid.Intersect`, `IntersectAllFaces`: geometry |
| `AnalysisModel.Select()` and `Select(true)`: update the analysis model (`Select(false)` reads) | `Operation.ObjectMatchesToFilter`; `Profile.ParseProfileString`, `FormatProfileString` |
| `Drawing.GetPlotFileName`: creates the `PlotFiles` folder in the model folder | `TeklaStructuresFiles.LocateFile`: searches, returns a path |

## No rollback, and one undo step

- An inserted object is in the model; a modified one is modified. Leaving out the commit takes
  neither back, and no API call does. Drawing and catalog changes are outside the model's undo.
- `model.CommitChanges()` closes the changes since the last commit into one undo step. **Nothing
  calls it for you.** A snippet that changed the model calls it itself, once, as its last step.
  Pass it a few words of description: the documentation marks the parameterless overload as
  deprecated since version 2018 **[lookup only]**.
- For the user to undo your work in one go, the whole change must be one run with one commit. A
  commit after every object, or a change spread over several runs that each commit, hands the user
  as many undo steps.
- **A run that changed nothing in the model does not commit.** That is every run that only read,
  and equally every run that only printed, only exported or made a report, only selected, or only
  ran the clash check. A commit there adds a step to the user's undo for nothing, and closes
  whatever else was pending into it **[inferred]**.
- A change that was never committed is still in the model: not drawn, in no undo step, and waiting
  for the next commit by anyone **[inferred]**. If a run changed the model and ended without
  reaching its commit, say so; do not leave it unmentioned.
- So: validate before the first write; keep a list of the GUIDs you create; if the run fails, return
  that list and say plainly what is now in the model.

## Never, unless the user asked for exactly this

- **Save** the model or a drawing. Saving makes everything permanent, the user's own experiments
  included. (`ModelHandler.IsModelSaved()` says whether there are unsaved changes **[lookup only]**:
  worth reading before a risky step, so you can warn the user.)
- **Open or close** a model. Closing discards unsaved work; opening replaces the session.
- **Run a macro** (`Operation.RunMacro`). A macro can do anything, saving included, unseen.
- **Run numbering**; update, issue or unissue drawings; lock or freeze them.
- **Change a catalog, an advanced option, or a process-wide setting.**
- **Send keystrokes or drive the application's windows.** A stray key can save or delete.
- **Overwrite or delete a file** you did not create in this conversation.

## The user's screen: put it back, if it was yours

Same rule for all three: note the state before, change, restore in a `finally`, verify, and report
if the restore failed.

**Selection.** Snapshot with the UI selector's `GetSelectedObjects()` and `SelectInstances = false`;
restore with `Select(snapshot, false)`; verify by comparing identities, not counts and not the
return value. If the user asked you to select something, the new selection is the result: leave it.

**Open drawing.** The editor holds one drawing. Opening one replaces the one that is open, and the
API can only close "the active drawing", whichever that is at that moment.

- If any drawing is open when you start, it is the user's. Do not open another; do not close it.
- If none is open and you open one, it is yours only while you can prove it. Before closing, ask
  which drawing is active and confirm it is the same one (`IsSameDatabaseObject`, or the drawing's
  GUID; chapter 4). Mark and name are not proof: two drawings can share them.
- If the active drawing is a different one, the user opened it while you worked. If the application
  says a drawing is open but not which, it may be theirs. In both cases close nothing and say so.
- One close attempt, without saving. Then look again; do not retry: by then the active drawing may
  be one the user has just opened.

**Work plane.** `SetCurrentTransformationPlane` changes the plane every later API read is expressed
in, and it can outlive your snippet: if the snippet fails before restoring it, every later
coordinate is silently shifted. Prefer to transform points instead (chapter 1). If you must change
it, read the current plane first and restore it in a `finally`. The on-screen work plane follows
only after `ViewHandler.RedrawWorkplane()` **[lookup only]**: do not call that unless moving the
user's work plane is the request.

## Printing with a drawing open: the classic way to lose work

**Why.** To print a drawing, the application opens it in the drawing editor, out of sight, and
closes it again. The editor holds one drawing. If the user has a drawing open, the print closes it
**[verified 2026]**. We know of nothing that saves it first, so assume that whatever the user drew
since their last save is lost. The print itself succeeds, so nothing in your result looks wrong. The
vendor documentation mentions the closing in one sentence on one overload.

**What to check.**

1. Before printing anything: `GetActiveDrawing() != null || IsAnyDrawingOpen()`. If true, print
   nothing and ask the user to save and close the drawing.
2. Before **each** drawing of a batch, check again, as late as possible. A batch runs for minutes
   and the user can open a drawing meanwhile. If one appears, stop there and report which drawings
   were not printed.
3. If the check itself fails, treat the editor as busy. Printing blind is the dangerous case.
4. After the batch, check once more. A drawing open now that was not before is reported, not closed.
   The same applies to anything else that opens a drawing, such as reading frame sizes.

## Files

- **The model folder is the application's working data**: databases (`.db1`), settings (`.ini`),
  catalogs, logs, `TeklaStructuresModel.xml`. Some have the same extensions an export can have.
  Never write over a file there that you did not write. Inside it, write only under the output
  subfolders the application itself uses (`PlotFiles`, `IFC`, `Reports`); elsewhere, write to a
  folder the user named.
- Do not replace an existing file unless the user asked; then move the old one aside, write,
  confirm, and only then delete it.
- Refuse a path that is, or passes through, a junction, symbolic link or mount point: your check of
  where the file lands compares spellings, and a link makes the spelling lie. (Windows marks
  cloud-sync folders similarly although they lead nowhere else: test for a real link.)
- Some calls write files you did not name: the IFC exporters' companion files, the clash check's
  `ClashCheck.history`, the `PlotFiles` folder. Know them (chapters 5, 7) and mention them.

## When something went wrong

Say it first and plainly: what is now in the model or on disk that was not asked for, which GUIDs,
which files, what is still selected or open. Offer the way back (the user's undo; deleting the
objects you list; closing a drawing without saving). No silent repair with more writes.
