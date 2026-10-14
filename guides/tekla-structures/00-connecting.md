# 0. Connecting

Use this chapter when you are setting the Dynamic Bridge up for Tekla Structures or the connection
fails.

Essentials: the Dynamic Bridge knows no application, so everything a snippet starts with comes from
a **profile** that you write on the user's machine; Tekla Structures must be running with a model
open, under the same Windows user as your tools; a stored profile and a running bridge are not yet
a connection, so prove it with a snippet.

("Profile" here is the Dynamic Bridge's description of an installed application. The steel sections
that Tekla Structures also calls profiles are in chapter 6.)

## The order

1. **`dotnet_list_profiles`.** If a profile for this installation exists and validates, use it: its
   usings, aliases, objects and notes say what your snippets start with. Go to step 5.
2. **`dotnet_scan_installation`** with part of the folder name (`tekla`) lists candidate folders
   under Program Files. Call it again with the folder of the version the user runs, to list the
   assemblies there: name, version, signer, and whether a documentation file sits beside each.
3. **Write the profile** from what the scan shows (next two sections) and store it with
   `dotnet_save_profile`. The answer names the signer of every referenced file: show the user who
   signed them before you rely on the profile, because that is whose code your snippets will call.
4. **`dotnet_ping`** confirms that the Dynamic Bridge is up for the profile. It loads nothing from
   Tekla Structures and says nothing about the model.
5. **Run the one-line proof** at the end of this chapter.

Storing a profile changes a file on the user's machine, so it follows the user's Guardrails
setting: stored in Allow changes, validated but not stored in Preview changes, refused in Read only
and Approve changes. Running a snippet needs Allow changes. Scanning and looking the API up do not
depend on the setting.

## Where the API files are

On Tekla Structures 2026 the API assemblies are in the installation's `bin` folder,
`C:\Program Files\Tekla Structures\2026.0\bin` **[verified 2026]**. Another version has another
version folder: let the scan find it, do not build the path yourself. The scan of `bin` is long,
about 340 assemblies, most of them not API **[verified 2026]**. These are the ones that matter:

| Assembly | Where | Documentation file | Holds | In the profile |
|---|---|---|---|---|
| `Tekla.Structures.dll` | `bin\Net48Runtime` | no | The core every other API assembly builds on: `Identifier`, geometry, settings. | Reference it. |
| `Tekla.Structures.Model.dll` | `bin` | yes | The 3D model, the selection, operations such as exports and reports. | Reference it. |
| `Tekla.Structures.Drawing.dll` | `bin` | yes | Drawings and everything on them. | Reference it. |
| `Tekla.Structures.Catalogs.dll` | `bin` | yes | Section profiles, materials, bolts, rebar, shapes, components. | Reference it. |
| `Tekla.Structures.Datatype.dll` | `bin` | yes | Typed values (distances, angles) and unit settings. | Reference it. |
| `Tekla.Structures.Analysis.dll` | `bin` | yes | The analysis model. Not covered by this guide. | Only if the task needs it. |

Locations and documentation files as scanned **[verified 2026]**.

- **`Tekla.Structures.dll` is not in `bin`.** It is in the sub-folder `Net48Runtime`
  **[verified 2026]**. So `Net48Runtime` goes into the profile as a probe sub-folder, and the file
  into the references under its relative path, `Net48Runtime\Tekla.Structures.dll`. A profile
  without it does not get as far as a connection **[inferred]**.
- **All six are signed by the application's publisher**, Trimble **[verified 2026]**. The profile is
  accepted only for an installation under Program Files whose referenced files carry a valid
  signature. A different signer on one of these files is a reason to stop and ask the user.
- **The core assembly has no documentation file.** A "docs" lookup for one of its types can come
  back empty while "types" and "members" still answer **[inferred]**.
- Leave everything else out: dialogs, user-interface and internal assemblies, third-party
  libraries, and anything with `Internal` in its name.

## What to put in the profile

| Part of the profile | Sensible content | Why |
|---|---|---|
| Installation folder | The `bin` folder the scan found. | Files are loaded only from there. |
| Probe sub-folders | `Net48Runtime` | Where the core assembly is. |
| Process name | `TeklaStructures` **[verified 2026]** | The Dynamic Bridge then reports whether the application is running, and refuses to run code when it is not. |
| References | The first five assemblies of the table above. | A profile with these five connected and read the model name **[verified 2026]**. |
| Usings | `Tekla.Structures`, `Tekla.Structures.Model`, `Tekla.Structures.Geometry3d`, `Tekla.Structures.Catalogs` | The names nearly every snippet needs, written short. |
| Alias for the drawing namespace | for example `TSD` for `Tekla.Structures.Drawing` | Never a using: Model and Drawing both declare `Part`, `Weld`, `Grid`, `ModelObject`, `Connection`, `Events` and more **[lookup only]**. With both as usings none of those names compiles. |
| Alias for the screen-side model namespace | for example `TSMUI` for `Tekla.Structures.Model.UI` | Never a using either: it has a `ModelObjectSelector` of its own, a different class from the one in the model namespace **[lookup only]**. |
| Catalog namespace | a using, or an alias such as `TSC` | Its documented type names do not collide with the model's **[lookup only]**, so a using works. An alias only keeps the origin of a type visible. |
| Objects | `model` of type `Tekla.Structures.Model.Model`; optionally `drawingHandler` of type `Tekla.Structures.Drawing.DrawingHandler` | The two ways in that most snippets need. |
| Notes | Units, where the commit belongs, what you learned. | The next reader of the profile has only this. |

- Each object is created once, with its parameterless constructor, and **kept between runs**.
  Everything else (a `CatalogHandler`, the screen-side selector, an `Events` object) you create in
  the snippet with `new`.
- Every snippet also gets `System`, `System.Collections`, `System.Collections.Generic`,
  `System.Linq` and `System.Text`. Anything outside those and the profile's own usings needs its
  full name (`System.Threading.ManualResetEvent`).
- The names are yours to choose. This guide writes `model` and `TSD` in the sense of the example
  below; if your profile names them differently, read the guide's names as yours.

## An example profile

This is an example to adapt, not a file to copy: the folder is that of one version on one machine,
and the id, the names and the aliases are choices. With these references and objects the connection
worked on Tekla Structures 2026 **[verified 2026]**.

```json
{
  "schema": 1,
  "id": "tekla-2026",
  "display_name": "Tekla Structures 2026",
  "runtime": "net48",
  "install": {
    "folder": "C:\\Program Files\\Tekla Structures\\2026.0\\bin",
    "probe_subfolders": ["Net48Runtime"],
    "process_name": "TeklaStructures"
  },
  "references": [
    "Net48Runtime\\Tekla.Structures.dll",
    "Tekla.Structures.Model.dll",
    "Tekla.Structures.Drawing.dll",
    "Tekla.Structures.Catalogs.dll",
    "Tekla.Structures.Datatype.dll"
  ],
  "framework_references": [],
  "usings": [
    "Tekla.Structures",
    "Tekla.Structures.Model",
    "Tekla.Structures.Geometry3d",
    "Tekla.Structures.Catalogs"
  ],
  "aliases": {
    "TSD": "Tekla.Structures.Drawing",
    "TSMUI": "Tekla.Structures.Model.UI"
  },
  "objects": [
    { "name": "model", "type": "Tekla.Structures.Model.Model" },
    { "name": "drawingHandler", "type": "Tekla.Structures.Drawing.DrawingHandler" }
  ],
  "notes": "Tekla Structures 2026. Tekla.Structures.dll is in the Net48Runtime sub-folder, the other API assemblies in bin. Drawing types are behind TSD because they clash with model type names; the screen-side selector is behind TSMUI. Nothing is committed for you: a snippet that changes the model calls model.CommitChanges once, at the end. Lengths are millimetres."
}
```

## Proving the connection

One line, which reads and changes nothing:

```csharp
result["connected"] = model.GetConnectionStatus(); result["model"] = model.GetInfo().ModelName;
```

`true` and the name of the model the user has open (it ends in `.db1`) is a working connection
**[verified 2026]**. Anything else is a failure, even when the snippet itself ran without an error.
If the profile has a `drawingHandler`, its `GetConnectionStatus()` answers for the drawing side.

## When the connection fails

| What you see | What it means | What to do |
|---|---|---|
| `dotnet_ping` or `dotnet_run_code` says the profile does not validate | A folder, reference or signature check failed. | The answer lists each error: correct the profile and store it again. |
| `dotnet_run_code` is refused because of the execution mode | The Guardrails setting is Read only or Approve changes. | Tell the user; the setting is theirs to change. |
| The snippet "succeeds" but nothing was run | The setting is Preview changes: compiled and checked only. | The same. |
| `host_not_ready` | An object of the profile could not be created, or an assembly could not be loaded: Tekla Structures is not running, or the references or the probe sub-folder are incomplete. | Start the application and open a model; check that `Net48Runtime` is a probe sub-folder and `Tekla.Structures.dll` a reference. |
| `compilation_error` on a name this guide uses | Your profile does not define that using, alias or object, or two usings declare the name. | Read `dotnet_list_profiles`; write the full type name, or correct the profile. |
| The snippet runs and `GetConnectionStatus()` is `false` | The API did not reach a model. | The three checks below. |

Three checks for a connection status of `false`, in this order:

1. **A model must be open.** Tekla Structures running without an open model is not enough: there is
   nothing to connect to.
2. **The same Windows user.** The application and the tools you call through must run under the
   same Windows user account. Working over remote desktop is supported.
3. **Restart the worker after the model was opened.** The worker is the running session that holds
   the profile's objects; they are created once and kept. A worker whose first connection failed
   does not recover on its own: the status stays `false` even after the user opens a model. Call
   `dotnet_ping` with its restart option, then run the proof again.

A session that is already running also keeps the profile it started with: after you store a changed
profile, the answer says so when one is found. Restart it the same way before you test the change.
