# 2. Reading the model

Use this chapter when you need to find model objects and read their properties: parts, bolts, welds,
connections, components, grids, pours, cuts, loads, reinforcement.

Essentials: lengths are mm; return GUIDs, not live objects; reading changes nothing, but it is slow
per object, so count round trips; a snippet that only reads does not call the model's commit.

`model` in this chapter is a connected `Tekla.Structures.Model.Model`, if your Dynamic Bridge
profile defines an object under that name. The short type names assume the profile's usings.
`dotnet_list_profiles` shows what yours defines (chapter 0); a full type name always works.

## Finding objects

The way in is `model.GetModelObjectSelector()`: the **database** selector, nothing to do with what
the user has selected on screen (chapter 7).

| You want | Call | Notes |
|---|---|---|
| One kind | `GetAllObjectsWithType(ModelObject.ModelObjectEnum.X)` | The normal case. |
| Several classes at once | `GetAllObjectsWithType(new[] { typeof(Part), typeof(Assembly) })` | Base classes work: `typeof(Part)` covers every part class. |
| Everything | `GetAllObjects()` | Includes welds, cuts, grids, tasks: keep what you need by class. |

**Every part.** The enum has no member that means "any part" **[lookup only]**, and asking for
`UNKNOWN` returns nothing **[verified 2026]**: it is not a wildcard. Two ways work:

- Take `GetAllObjects()` and keep what is a `Part` (`en.Current as Part`). The simple way on a small
  model.
- Ask for each concrete kind in the table below (`BEAM`, `POLYBEAM`, `CONTOURPLATE` ...) and add the
  answers together.

To filter by a property value: enumerate the kind, read the property on each object, compare in C#.
Compare numbers by value with a small tolerance (a length held as 6000.0000001 must match 6000) and
text without regard to case. When you query several kinds in turn the same object can come back
twice: de-duplicate by GUID.

## The kinds

One word in the user's mouth is often several enum members. Ask for all of them.

| User says | Enum members | Class | Where its name lives |
|---|---|---|---|
| beam, column | `BEAM` | `Beam` | `Name`, `Profile`, `Material`, `Class` |
| polybeam, plate, bent plate | `POLYBEAM`, `CONTOURPLATE`, `BENT_PLATE` | `PolyBeam`, `ContourPlate`, `BentPlate` | as a beam |
| part | the four above, plus `LOFTED_PLATE`, `SPIRAL_BEAM`, `BREP` (items) | `Part` subclasses | as a beam |
| bolt | `BOLT_ARRAY`, `BOLT_CIRCLE`, `BOLT_XYLIST` | `BoltGroup` subclasses | `BoltStandard`, `BoltSize` |
| weld | `WELD`, `LOGICAL_WELD`, `POLYGON_WELD` | `BaseWeld` subclasses | `MainObject`, `SecondaryObject`, sizes |
| assembly | `ASSEMBLY` | `Assembly` | `Name` |
| rebar | `SINGLEREBAR`, `REBARGROUP`, `REBAR_SET` (also `REBARMESH`, `REBARSTRAND`) | `Reinforcement` subclasses; `RebarSet` is a class of its own | on `Reinforcement`: `Name`, `Grade`, `Class` (number), `Father` |
| connection, detail, seam | `CONNECTION`, `DETAIL`, `SEAM` | `Connection`, `Detail`, `Seam` | `Name`, `Number`, `Class` (number) |
| component, custom part | `COMPONENT`, `CUSTOM_PART` | `Component`, `CustomPart` | `Name`, `Number` |
| grid | `GRID`, `GRIDPLANE` | `Grid`, `GridPlane` | `Name`; a grid plane has `Label` and `Parent` |
| pour | `POUR_OBJECT`, `POUR_BREAK` | `PourObject` | `PourNumber`, `Class` (number) |
| cut | `BOOLEANPART`, `CUTPLANE`, `FITTING` | `BooleanPart`, `CutPlane`, `Fitting` | none of its own; see below |
| load | `LOAD_POINT`, `LOAD_LINE`, `LOAD_AREA`, `LOAD_UNIFORM`, `LOAD_TEMPERATURE` | `Load` subclasses | `Group.GroupName` |
| surface treatment | `SURFACE_TREATMENT` | `SurfaceTreatment` | `Name`, `Material`, `Class`, `Father` |

- The enum differs between versions: turn a name into a value with `Enum.TryParse` and report the
  names that do not exist.
- **Cuts** are objects of their own, attached to the part they cut (`Father`). A part cut
  (`BooleanPart`) holds the cutting solid as `OperativePart`, and that is where a name, profile and
  material are found. Cut planes and fittings have only the parent.

## Reading one object by GUID

```csharp
var identifier = model.GetIdentifierByGUID(guid);
var obj = identifier.IsValid() ? model.SelectModelObject(identifier) : null;
```

Three outcomes, and they must stay three:

| What happened | Meaning |
|---|---|
| `GetIdentifierByGUID` throws `ArgumentException` or `FormatException` | The text is not a GUID at all. |
| The identifier is not valid (its `ID` is 0) | No such object. Also what a **deleted** object's GUID gives. **[verified 2026]** |
| The identifier is valid but `SelectModelObject` returns `null` | The object exists and its data did not load. Retry; do not report it as missing. |

The application's reports and IFC lists print GUIDs with the prefix `ID` (38 characters): strip it
first. For many GUIDs use `model.FetchModelObjects(guids, false)`: one round trip. It returns only
the objects that exist and silently leaves the others out, so the misses are whatever is absent
from the answer. If the call throws half-way, discard its answer and look the ids up one by one: a
half-read answer would report existing objects as missing.

## Reading a part

Once loaded, these are free: `Name`, `Profile.ProfileString`, `Material.MaterialString`, `Class`,
`Finish`, and for a beam `StartPoint` and `EndPoint`. Read each in its own `try`: one bad catalog
reference must not hide the other fields, and a failed field is reported as failed, not as empty.
Computed values are report properties. These names are known to work on parts:

| Name | Type | Meaning |
|---|---|---|
| `WEIGHT` | double | kg **[inferred]**: check one known part |
| `LENGTH` | double | mm |
| `PROFILE`, `MATERIAL`, `NAME`, `CLASS` | string | as shown in the application |
| `PART_POS`, `ASSEMBLY_POS`, `REBAR_POS` | string | part mark (`P/12`), assembly mark (`A/3`, also on an `Assembly`), rebar mark |

- Report property names are the application's template field names. There are hundreds and the
  lookup does not list them: try a name on one object before using it in a loop. A mark that is not
  current ends in `(?)`; `P/0(?)` means never numbered (chapter 7).
- UDAs: `GetUserProperty(name, ref value)`. Which exist depends on the office's setup;
  `GetAllUserProperties(ref hashtable)` returns what one object carries **[lookup only]**.
- When a loop is slow, try the bulk reads **[lookup only]**: `GetStringReportProperties` (several
  names, one object) and `model.GetReportPropertyStr` (one name, many identifiers).

## Geometry, and what cannot be read the way you expect

- `part.GetSolid()` gives `MinimumPoint`, `MaximumPoint`, and enumerators of faces, loops and
  vertices (`Tekla.Structures.Solid`). Check `solid.IsValid()`: a part can have no usable solid.
  Walking faces is slow, so bound it (faces, vertices, seconds per part). We know no cheap way to
  mesh bolts, welds and reinforcement.
- Coordinates come in the current transformation plane, not necessarily global: transform with the
  plane's `TransformationMatrixToGlobal` (chapter 1).
- An identity-only object (enumerator with `SelectInstances = false`) has an empty GUID and empty
  part fields. `Select()` it, or ask the model for its GUID. Report properties still work.
- `false` from a report property means wrong name, wrong type, or not applicable: never which.
- Material strengths are not in the API (chapter 6). A part does not know its drawings: they are a
  different API (chapter 4).
