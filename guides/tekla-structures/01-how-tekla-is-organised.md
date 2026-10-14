# 1. How Tekla Structures is organised

Use this chapter when you are about to write your first snippet, or when a snippet is correct C# but
slow, returns empty values, or gives coordinates you do not recognise.

Essentials: lengths are mm; objects are identified by GUID; nothing is committed or saved for you.

`model` in this chapter is a connected `Tekla.Structures.Model.Model`, if your Dynamic Bridge
profile defines an object under that name. The short type names assume the profile's usings.
`dotnet_list_profiles` shows what yours defines (chapter 0); a full type name always works.

## The client and the running program

The Open API is a client library. Your snippet runs outside Tekla Structures and talks to the
running program over a local channel.

- **A model must be open.** Without one there is nothing to connect to.
  `model.GetConnectionStatus()` and a non-empty `model.GetInfo().ModelPath` both say "connected".
  If they do not, see chapter 0.
- **One program, one model.** A connection reaches one running Tekla Structures and the model open
  in it. With two of its windows open you do not choose which answers **[inferred]**: have one
  closed.
- **One thread.** The API is not thread-safe. Call it from the thread your snippet runs on, never
  from a task, a timer or an event handler.
- **The application must be idle.** While a command is running or a dialog is open, calls fail,
  return `false`, or return nothing. The user must finish or cancel the command (Esc).

| Namespace | Entry point | What it reaches |
|---|---|---|
| `Tekla.Structures.Model` | `Model` | The 3D model: parts, bolts, welds, components, grids, phases. |
| `Tekla.Structures.Model.UI` | `ModelObjectSelector`, `ViewHandler` | What the user sees: selection, views. |
| `Tekla.Structures.Model.Operations` | static `Operation` | Numbering state, reports, exports. |
| `Tekla.Structures.Drawing` | `DrawingHandler` | Drawings and everything on them. |
| `Tekla.Structures.Catalogs` | `CatalogHandler` | Profiles, materials, bolts, rebar, shapes, components. |

Model and Drawing both declare `Part`, `Weld`, `Grid`, `ModelObject` and `Events`, among others:
keep the drawing types behind an alias. Which namespaces a snippet can use by short name, and under
which alias, is decided by your profile. Namespaces with `Internal` in the name are plumbing:
unsupported, different in every version. The analysis API is not covered by this guide.

## Objects, identity, and what "Select" means

Everything in the model derives from `ModelObject`. The kind is a class (`Beam`, `ContourPlate`,
`BoltArray`, `Weld`, `Connection` ...) and also a member of the enum `ModelObject.ModelObjectEnum`.
A column is a `Beam`. Chapter 2 has the table of kinds.

Each object has an `Identifier`: a permanent `GUID`, and an `ID` number valid for this session only.
`model.GetIdentifierByGUID(text)` and `model.GetGUIDByIdentifier(identifier)` convert. Some sources
give identifiers with a number and an **empty GUID** (the drawing API, and enumerators in
identity-only mode): ask the model for the GUID before returning it.

An object in your code is a **copy**. Reading its properties costs nothing; setting them changes
nothing in the model. Four calls cross to the application, each returning `bool` with no reason
attached: `Select()` refreshes the copy from the model (a read: nothing to do with the selection on
screen); `Insert()` creates the object, `Modify()` writes the copy over it, `Delete()` removes it,
each at once.

## Enumerators

Lists come back as enumerators, always with the same shape:

```csharp
var en = model.GetModelObjectSelector().GetAllObjectsWithType(ModelObject.ModelObjectEnum.BEAM);
while (en.MoveNext()) { var beam = en.Current as Beam; /* read beam */ }
```

- `GetSize()` gives the count without walking the list. By default each step loads the object: one
  round trip, so walking 10,000 objects is slow.
- `SelectInstances = false` skips the load. `Current` is then identity-only: the right class, a
  numeric id, an **empty GUID**, and empty class properties (`Name`, `Profile` ...). Use it to
  count, to pass objects on (to a selection, a clash check), or to ask by identity: report
  properties and the numbering check work on such an object.
- A kind with no objects gives an empty enumerator, never `null` **[verified 2026]**. `null` means
  the query failed.
- The order is not sorted by anything useful. It holds between two calls only while nobody edits the
  model.

## Commit and undo

`Insert`, `Modify` and `Delete` change the model immediately. `model.CommitChanges()` does two other
things: it closes everything since the last commit into **one undo step**, and it redraws the views.

- **Nothing is committed for you.** The Dynamic Bridge runs your snippet and nothing more. A snippet
  that changed the model calls the commit itself, **once, at the end**. Without it the change is in
  the model, but not drawn and in no undo step.
- **A snippet that only reads does not commit.** A commit there closes whatever is pending into an
  undo step of your making, and it is one more step in the user's undo **[inferred]**.
- **One run, one commit, one undo step.** Every extra commit splits the user's undo into more steps.
  So does a change spread over several runs that each commit.
- **Give the commit a description.** The documentation marks the parameterless `CommitChanges()` as
  deprecated since version 2018; the overload that takes a message string carries no such mark
  **[lookup only]**. Pass a few words that say what the run did.
- **There is no rollback.** Not committing does not take an inserted object back: it is in the
  model, just not yet drawn and not yet in an undo step. The public API has no undo call
  **[lookup only]**; only the user can undo, in the application. And a commit is not a save.

## Three kinds of properties

| Kind | How you read it | Notes |
|---|---|---|
| Class properties | `part.Name`, `part.Profile.ProfileString`, `part.Class` | What the user typed. Free once loaded. |
| Report properties | `GetReportProperty(name, ref value)` | What the application computes: `WEIGHT`, `LENGTH`, `PART_POS`. A round trip each. |
| User-defined attributes (UDAs) | `GetUserProperty(name, ref value)` | Fields the office added. A round trip each. |

Report and user properties have overloads for `string`, `double` and `int`. The return value says
whether the property exists **with that type**; the value comes back through a `ref` argument, so
declare the variable first.

Names are case-sensitive. If you do not know a property's type, ask for a string first; if that
fails or comes back empty, ask for a double, then an int. An exception is a failed read, not a
missing property: report the two differently.

## Units and coordinates

Lengths are mm, also in catalogs and on paper. Coordinates come back in the **current transformation
plane** (the API's side of the work plane), which is not always the global origin. For global
coordinates, transform with
`model.GetWorkPlaneHandler().GetCurrentTransformationPlane().TransformationMatrixToGlobal`.

Do not move the plane to get local coordinates: the setting can outlive your snippet and shift every
later read. Transform the point instead, with
`MatrixFactory.ToCoordinateSystem(part.GetCoordinateSystem()).Transform(point)`.

## Phases, assemblies, cast units

- **Phases.** `model.GetPhases()` is a non-generic collection of `Phase` (`PhaseNumber`,
  `PhaseName`, `IsCurrentPhase` as an int); `model.GetInfo().CurrentPhase` is the current number.
- **Assemblies.** A part belongs to one: `part.GetAssembly()`. An assembly carries the assembly mark
  (`ASSEMBLY_POS`); a part carries its part mark (`PART_POS`) and its assembly's mark.
- **Cast units.** The kind enum has no cast-unit member **[lookup only]**; concrete cast units come
  back as `Assembly` objects **[inferred]**.

## Cost

What costs time: an enumerator that loads each object; a report property or UDA per object;
`SelectModelObject` in a loop (use one `model.FetchModelObjects` call); `GetSolid()` and its faces.
`GetSize()` and identity-only enumeration are nearly free. Give every loop a time budget and a
limit. When it runs out, return what you have, say that the answer is incomplete and where to
continue, and label any total as a lower bound.
