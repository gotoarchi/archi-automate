# 9. Traps

Use this chapter when a snippet compiled and ran but the answer is `true` with nothing done, `null`,
zero, empty, slow, or simply not what Tekla Structures shows. Find the symptom, read the cause.

Essentials: lengths are mm; ids are GUIDs; nothing is committed or saved for you; a `true` from the
API is a claim, not evidence.

`model` and the short type names in this chapter exist only if your Dynamic Bridge profile defines
them. `dotnet_list_profiles` shows what yours defines (chapter 0); a full type name always works.

## Running, compiling, connecting

| Symptom | Cause | What to do |
|---|---|---|
| The run is refused because of the execution mode | The user's Guardrails setting is Read only or Approve changes. | Tell the user what the snippet would do; the setting is theirs. |
| The answer is fine but nothing happened and nothing came back | The setting is Preview changes: compiled and checked, not run. | The same. |
| `not_allowed`, naming a rule | The code-safety rules: file, process, network or reflection access in the snippet. | Do it another way or ask the user; do not work around it. |
| `model`, `TSD` or another name this guide uses does not compile | Your profile does not define that object, alias or using. | Use the names from `dotnet_list_profiles`, or full type names. |
| `Part`, `Weld`, `Grid`, `ModelObject` or `Events` is ambiguous | Model and Drawing both declare them. Likewise `Task`, if `System.Threading.Tasks` is among the usings. | Keep Drawing behind an alias (chapter 0); write `Task` in full. |
| `ModelObjectSelector` has no `GetSelectedObjects` | There are two: the database one (from `model`) and the UI one. | Selection is `Tekla.Structures.Model.UI.ModelObjectSelector`. |
| `host_not_ready` | Tekla Structures is not running, or the profile's references or probe sub-folder are incomplete. | Chapter 0. |
| `GetConnectionStatus()` is `false`, also after the user opened a model | No model was open when the profile's objects were first created, and they are kept; or a different Windows user. | Restart the profile's worker with `dotnet_ping` and its restart option (chapter 0). |
| Calls fail or return nothing now and then | The application is busy: a command is running or a dialog is open. | Retry once; ask the user to finish the command. |

## Reading the model

| Symptom | Cause | What to do |
|---|---|---|
| `GetAllObjectsWithType` with `UNKNOWN` returns nothing | `UNKNOWN` is not a wildcard, and no member means "any part". | `GetAllObjects()` and keep the `Part`s, or ask per concrete kind (chapter 2). |
| GUIDs are all zeros | Identity-only objects, or identifiers from the drawing API. | `model.GetGUIDByIdentifier(identifier)`, or `Select()` the object. |
| Name, profile and material are empty on every object | Enumerator with `SelectInstances = false`. | `Select()` the ones you keep, or read report properties. |
| `SelectModelObject` returns `null` | No such object, or its data did not load. | `identifier.IsValid()`: invalid means missing, valid means retry. |
| No exception for an unknown or deleted GUID | `GetIdentifierByGUID` returns an invalid identifier instead. | Test `IsValid()`. |
| A report property returns `false`, or an empty string for a number | Wrong name (case matters), wrong type overload, or not applicable. | Try string, then double, then int, on one object. |
| Columns, bolts or welds are missing | A column is a `Beam`; bolts and welds are three enum members each. | Ask for every member (chapter 2). |
| Coordinates are offset or rotated | They are in the current transformation plane. | Transform to global (chapter 1). Do not move the plane. |

## Changing the model

| Symptom | Cause | What to do |
|---|---|---|
| `Insert()` returned `false` | The application refused; no reason is given. | Check inputs against the catalog; check the application is idle. |
| The object is in the model although you "did not commit" | Inserts take effect at once. | There is no dry run (chapter 3). |
| The change is in the model but not on screen, and the user's undo does not reach it | The snippet never called the commit, and nothing commits for you. | End every changing snippet with one `model.CommitChanges(...)` (chapter 3). |
| The user has to undo many times for one request | A commit per object, or one per run of a change split over several runs. | One run, one commit, at the end. |
| A value the user just changed went back | `Modify()` from an old copy **[inferred]**. | Load, change and modify in the same run. |

## Drawings, printing, IFC, reports

| Symptom | Cause | What to do |
|---|---|---|
| View width and height are 0; `GetView()` throws | The drawing is closed. | Report "unknown"; read container by container (chapter 4). |
| A type filter throws | `SectionMarkBase` is not accepted as a filter. | Name the concrete classes, or enumerate all and test. |
| No dimensions found on the sheet | The sheet holds the dimension set, not the dimensions. | Go through the set's `GetObjects()`. |
| `GetActiveDrawing()` is `null` but printing closed something | `IsAnyDrawingOpen()` was `true`. | Test both, always. |
| `CloseActiveDrawing` returned `false`, yet the drawing closed | The return value is unreliable. | Look at the editor state; do not call it twice. |
| `PrintDrawing` returned `true`, no file | The folder did not exist or was not writable. | Create the folder; confirm the file on disk. |
| The user's open drawing closed during a print | Printing uses the editor. | Check before every print (chapter 8). |
| The PDF is 1 to 3 % too large | Automatic scaling fills the paper. | Fixed scale 1.0. |
| The IFC2x3 export throws `FormatException` | Empty base-point text. | Pass the all-zero GUID. |
| `CreateReport...` returned `false` | Template not found (wrong name, `.rpt` included, three characters or fewer), or the folder is missing. | Find the template file first; create the folder. |
| The report file is empty or half written | Creation is asynchronous. | Wait until it holds still (chapter 5). |

## Catalogs, selection, numbering, clash check

| Symptom | Cause | What to do |
|---|---|---|
| A profile is not found although Tekla Structures offers it | It is parametric: a prefix plus dimensions. | Search both catalogs; build the string from the pattern. |
| A profile parameter is a huge integer, or 25 properties are mostly 0 | Wrong value field for its unit type; unset reads as 0. | Choose the field by unit type; leave zeros out (chapter 6). |
| A component has no name; no yield strength anywhere | System components have a number only; strengths are not in the API. | Use `UIName` and `Number`; say strengths are unavailable. |
| `Select` returned `false` | A command is running or a dialog is open. | The user finishes or cancels it. |
| Fewer objects selected than given, no error | Hidden or filtered-out objects are dropped. | Read back, compare by identity, report the missing. |
| `IsNumberingUpToDate` throws `ArgumentException` | The kind is not numbered. | Catch it; report "not a numbered kind". |
| `RunClashCheck` returned `false` | Another check is running, or a command is active. | Wait; ask the user to close the Clash Check Manager. |
| More clashes than the application counted | Another check's events are mixed in. | Mark the result incomplete; run again alone. |
| Clash handlers still fire after `-=` | The handler was a lambda written after `+=`: `-=` with a new lambda removes nothing. | Keep each delegate in a variable of its type; then `UnRegister()` (chapter 7). |
| `Overlap` is 0 on a real clash | For plain clashes the value is not given **[verified 2026]**. | Report it as not given; there is no clash volume. |
| The parts on a clash have no name or profile | They carry identity only. | Load them by GUID (chapter 7). |
