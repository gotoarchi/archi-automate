# 5. Printing and exporting

Use this chapter when you print drawings to PDF, export IFC, or create a report.

Essentials: paper and model lengths are mm; none of these change the model, so none of them calls
the model's commit; all of them write files, and **printing closes whatever drawing the user has
open**; nothing is saved for you.

`model` in this chapter is a connected `Tekla.Structures.Model.Model`, `handler` a
`Tekla.Structures.Drawing.DrawingHandler`, and `TSD` an alias for the drawing namespace. Each exists
only if your Dynamic Bridge profile defines it, perhaps under another name. `dotnet_list_profiles`
shows what yours defines (chapter 0); a full type name always works.

## Rules for all three

1. **Always pass a full path.** A relative path is resolved by the application, against the model
   folder or a folder named in an advanced option, not against anything you can see.
2. **The folder must exist first.** The application does not create it: printing then reports
   success and writes nothing, and a report fails.
3. **Do not replace a file unless the user asked.** Your folder and file name are a guess. When
   asked: move the old file aside under a name nothing else can have, write, confirm the new file,
   then delete the old one; move it back if the write failed. Never delete first.
4. **The API's `true` is not evidence.** Evidence is a new file on disk that has stopped changing.
   Note the file's size and write time **before** the call, to tell a new file from the old one.
5. **File work inside a snippet can be refused.** The Dynamic Bridge's code-safety rules do not let
   a snippet touch files itself by default: the answer is `not_allowed` and names the rule. Tekla
   Structures writing the PDF, the IFC file or the report is not affected. Create folders and check
   files with a file tool of your own, or ask the user to look; do not work around a refusal.
6. **Stay out of the model folder's own files** (chapter 8). Its `PlotFiles`, `IFC` and `Reports`
   subfolders are where the application itself puts these outputs: the safe defaults.
7. **Say what was written.** A file on the user's disk is outside every undo. Tell the user the
   full path of each file before the run, and again, confirmed, after it.

## Printing drawings to PDF

`DrawingHandler.PrintDrawing(drawing, attributes, outputFile)` with a `DPMPrinterAttributes`. With
output type PDF, the application writes the PDF itself: no printer driver.

```csharp
var attributes = new TSD.DPMPrinterAttributes { OutputType = TSD.DotPrintOutputType.PDF };
attributes.ScalingMethod = TSD.DotPrintScalingType.Scale; attributes.ScaleFactor = 1.0;
bool accepted = handler.PrintDrawing(drawing, attributes, fullPdfPath);
```

**A print is an ordinary run that writes a file.** It needs the Guardrails setting every snippet
needs in order to run (Allow changes) and nothing more. For the user that means: a PDF appears
where you said, the model and the drawing are as they were, and no undo takes the file away. So
name the file beforehand, never replace one unasked, do not commit, and report the path.

**Before each print, check that no drawing is open**
(`GetActiveDrawing() != null || IsAnyDrawingOpen()`). If one is, print nothing and ask the user to
save and close it: printing uses the drawing editor, which holds one drawing, so the user's is
closed **[verified 2026]** and we know of nothing that saves it first. In a batch, check again
before every drawing; if the state cannot be read, treat it as open. After the batch, a drawing open
that was not before is reported, not closed. Chapter 8 has the reasoning.

What is known **[verified 2026]**:

- A closed drawing prints as it is stored. Its modification date, issue flags and up-to-date status
  are unchanged afterwards.
- `PrintDrawing` answers `true` even when it wrote nothing (a folder that does not exist). With
  paper size and orientation left automatic, the application picks the paper that fits the sheet.
- **Scaling.** The automatic setting enlarges the sheet to fill that paper: 1.034 for an 804 x 557
  mm sheet on A1, 1.010 for 841 x 594. A view marked 1:50 then no longer measures 1:50. A fixed
  scale of 1.0 prints exactly 1:1. Use it unless the user asks to fit.
- **The PDF's title is the file name it is printed to.** Print straight to the final name. A print
  takes a few seconds per sheet: keep a batch small (about 20) and time-boxed.
- In every measured run the call returned only after the file was written. Still confirm: it
  exists, is not empty, and can be opened exclusively. Nothing a second after the call: nothing is
  coming.

**File names.** `drawing.GetPlotFileName(true)` returns the name the office's own plot-file rule
gives (the `XS_DRAWING_PLOT_FILE_NAME_...` advanced options), revision mark included: use it, so
files get the names the team expects. Asking for it **creates the `PlotFiles` folder** in the model
folder as a side effect **[verified 2026]**. Replace characters Windows does not allow, fall back to
mark and name when the rule gives nothing, and make names unique within a batch.

**Failures.** An exception carries the application's own reason (for example, that the drawing must
be updated first). `false`, or `true` with no file, means refused or not written: locked, not up to
date, failing to open, or a folder that cannot be written.

## Exporting IFC

Static methods on `Tekla.Structures.Model.Operations.Operation`. There are two different exporters,
and their options do not mix.

| | IFC4 and IFC4.3 | IFC2x3 |
|---|---|---|
| Method | `CreateIFC4ExportFromAll`, `CreateIFC4ExportFromSelected` | `CreateIFC2x3Export` (one entry point) |
| Whole model or selection | chosen by the method | `flags.ExportAllObjects` (`false` = the selection) |
| Schema | `IFCExportVersionEnum`: `IFC4`, `IFC43` | fixed |
| View | `REFERENCE_VIEW`, `DESIGN_TRANSFER_VIEW` | `COORDINATION_VIEW_20`, `SURFACE_GEOMETRY`, `STEEL_FABRICATION_VIEW` |
| Flags class | `Operation.IFCExportFlags` | `Operation.IFC2x3ExportFlags` |
| Assembly flag is spelled | `IncludeAssemblyHierarchy` | `IncludeAssemblyHiearachy` (as in the API) |
| "No base point" is passed as | an empty string | the all-zero GUID as text |
| Log written beside the file | `.htm` | `.log` |

- **Origin** is `Operation.ExportBasePoint`: `GLOBAL` (model origin), `WORK_PLANE`, or `BASE_POINT`
  with the GUID of a base point defined in the model. **The IFC2x3 base-point argument is parsed
  even when unused**: an empty string throws `FormatException`, `null` throws
  `ArgumentNullException`.
- For IFC4 the layer and colour arguments are text: `"__Name__"` (layers by part name) and
  `"ByObjectClass"` (colours by class) work. Empty lists are accepted for the attribute-override and
  property-set file arguments.
- Flags switch content on and off: bolts, welds, grids, rebars, surface treatments, pours, assembly
  hierarchy. **Exporting pours instead of parts breaks the link to the model**: with pours off,
  every exported element is a model object with the GUID the rest of the API uses.
- "Selected" means the selection in the model view (chapter 7): check that it is not empty. The call
  returns when the export is done; allow a few seconds, then confirm the file. A very large model
  takes minutes.
- **Companion files are written beside the `.ifc` without asking**, with the same base name: the
  log, a `.partReport` (IFC4), and a script `ifc4export.js` in the same folder. Existing ones are
  replaced silently. Give each export its own file name or folder.
- The log lists what was skipped. When an export fails with no reason, the application's session
  log (under `logs` in the model folder) has it. `model.GetInfo().ModelName` ends in `.db1`.

## Creating reports

`Operation.CreateReportFromAll(template, fileName, title1, title2, title3)` and
`CreateReportFromSelected(...)` run a report template over the model or the selection.

- **Pass the template's name**: the `.rpt` file name **without** the extension, longer than three
  characters. With the extension, or with a name that does not exist, the call answers `false` and
  nothing else.
- **So find the template first.** Templates are `.rpt` files in the folders the application
  searches, in this order, first match wins: `XS_TEMPLATE_DIRECTORY`, the model folder,
  `XS_PROJECT`, `XS_FIRM`, `XS_TEMPLATE_DIRECTORY_SYSTEM`, `XS_SYSTEM`. Read each with
  `TeklaStructuresSettings.GetAdvancedOption(name, ref text)`. A value can hold many folders
  separated by `;` (`XS_SYSTEM` about two dozen); relative entries are relative to the model folder.
- **Template names differ between environments.** `Material_List_by_Size` ships with the default
  2026 environment; `Assembly_List` does not **[verified 2026]**. When the user's name is not there,
  list the closest and ask. Names containing `-----` are headings, not reports.
- A template whose name carries a format (`...(HTML).html.rpt`) writes that format. All others write
  plain text, conventionally `.xsr`. Never write a report to `.rpt`: that replaces a template.
- **Creation is asynchronous**: the call returns before the file is complete. Wait for the file to
  appear or change (allow a minute), then for it to hold still: the same size and write time on two
  looks a moment apart, and openable. An **empty** file must stay empty for a couple of seconds
  before you believe it: the application creates the file first and fills it afterwards.
- Plain reports are in the Windows ANSI code page, without a byte-order mark. HTML reports have an
  empty line after every line. Marks in a report are only right when numbering is up to date.
