# 4. Drawings

Use this chapter when you list drawings and their status, read what is on a drawing (views, marks,
dimensions, texts), or create and edit drawings. For printing see chapter 5.

Essentials: paper lengths are mm; model objects are identified by GUID, but the drawing API hands
out numbers, so convert; nothing is committed or saved for you. **The drawing editor holds one
drawing and it belongs to the user**: do not open or close drawings to read them.

`model` in this chapter is a connected `Tekla.Structures.Model.Model`, and `handler` a
`Tekla.Structures.Drawing.DrawingHandler`. Your Dynamic Bridge profile decides whether objects of
those types exist, and under which names, and under which alias the drawing namespace is reached.
`dotnet_list_profiles` shows what yours defines (chapter 0); a full type name always works.

## The way in, and the list

Take the `DrawingHandler` your profile defines, or create one with `new`, and check
`GetConnectionStatus()`: `false` means the drawing API is not answering (no model open, or the
application is showing a dialog). `GetDrawings()` enumerates every drawing in the model, in no
particular order: sort yourself. The class of a drawing is its type:

| Class | Letter (`DrawingTypeStr`) | Documents | Link to the model |
|---|---|---|---|
| `GADrawing` | G | general arrangement | none |
| `AssemblyDrawing` | A | one assembly | `AssemblyIdentifier`, `SheetNumber` |
| `SinglePartDrawing` | W | one part | `PartIdentifier`, `SheetNumber` |
| `CastUnitDrawing` | C | one cast unit | `CastUnitIdentifier`, `SheetNumber` |
| `MultiDrawing` | M | a multidrawing | none |

All of this reads from the enumeration without opening anything:

- `Mark` (for example `[12]`), `Name`, `Title1` to `Title3`. An unset title is a single space.
- `IsIssued`, `IsIssuedButModified`, `IsReadyForIssue`, `IsLocked`, `IsFrozen`, `IsMasterDrawing`,
  `IsLockedBy`, `IsReadyForIssueBy`; and `UpToDateStatus`: `DrawingIsUpToDate` when current,
  otherwise the reason (`PartsWereModified`, `AllPartsDeleted` ...; the enum is in the lookup).
- `CreationDate`, `ModificationDate`, `IssuingDate` ("never" is `DateTime.MinValue`), and
  `Layout.SheetSize` (`Width`, `Height`; read it in a `try`, it can fail on its own).
- The identifiers on part-based drawings carry only the session number. Convert with
  `model.GetGUIDByIdentifier(identifier)`; an empty answer means the object no longer exists.

**A drawing has no public id.** Type + mark + name is what a person calls a drawing, but it is
**not** an identity: two drawings can share it, and if it matches more than one you must stop and
ask. `drawing.IsSameDatabaseObject(other)` compares two references **[lookup only]**. On 2026 the
drawing's GUID can be read through `DatabaseObjectExtensions.GetIdentifier` in
`Tekla.Structures.DrawingInternal`; that is internal: unsupported, and possibly absent in another
version (`MissingMemberException`, `TypeLoadException`). The GUID survives saving, reopening and
renaming; a drawing deleted and created again gets a new one.

**Which drawing is open.** `GetActiveDrawing()` returns the drawing in the editor, or `null`.
`IsAnyDrawingOpen()` can be `true` while `GetActiveDrawing()` is `null`. Either means "open": test
`handler.GetActiveDrawing() != null || handler.IsAnyDrawingOpen()`.

## Reading what is on a drawing

**A closed drawing reads as it is stored** **[verified 2026]**: its views and every object on them,
marks with their text, dimensions with their values, texts. Nothing needs opening. If the user has
that drawing open, you read the open, possibly unsaved, state.

`drawing.GetSheet()` is the sheet (a `ContainerView`); `sheet.GetAllViews()` yields the views (keep
the ones that are `View`). Then read container by container: `sheet.GetObjects()` gives only what
sits directly on the sheet, `view.GetAllObjects()` everything in that view. Both take an optional
`Type[]` filter, which is much faster on a big drawing: marks, dimensions and texts alone read far
quicker than every object.

| You want | Type | Members |
|---|---|---|
| View | `View` | `Name`, `ViewType` (one value is spelled `_3DView`), `Attributes.Scale` (50 means 1:50) |
| Model object shown | drawing `ModelObject` (`Part`, `Bolt`, `Weld` ...) | `ModelIdentifier` |
| Part or bolt mark | `Mark` | `Attributes.Content`: an element tree, see below |
| Section or detail mark | `SectionMark`, `CurvedSectionMark`, `DetailMark` | `Attributes.MarkName` |
| Weld mark, level mark | `WeldMark`, `LevelMark` | `ModelIdentifier`, `ModelObjectIdentifier` |
| Straight dimension | `StraightDimension` | `Value` (text as drawn), `StartPoint`, `EndPoint` |
| Angle dimension | `AngleDimension` | `GetAngle()` (degrees) |
| Free text | `Text` | `TextString`, `InsertionPoint` |

The types in this table are those of the drawing namespace: write them through your profile's alias
for it, or in full. `Part`, `Weld` and `ModelObject` written short mean the model's classes.

- **Mark text is a tree**, not a string. Walk the `ContainerElement`: `TextElement.Value` is literal
  text; `PropertyElement.Value` is a field already resolved for this object (the profile field reads
  `HEA300`); `UserDefinedElement.Value`; `SpaceElement` and `NewLineElement` are white space; a
  nested `ContainerElement` is a frame, recurse; `SymbolElement` is a graphic.
- **What a mark points at:** `mark.GetRelatedObjects()`, keeping the drawing `ModelObject`s.
- **A dimension's measured length** is the distance from `StartPoint` to `EndPoint` (mm). `Value` is
  the text on paper and follows the dimension's unit and format settings.
- **On the sheet the dimension set is the direct child**, not the dimensions: reach them through the
  set's `GetObjects()`.
- Skip anything that is a `ViewBase`, and wrap each object's read in a `try`. Large drawings take
  minutes: time-box the walk, stop between views, report which view to continue from.

**What does not work on a closed drawing:**

- **A view's frame size is 0 x 0.** `Width` and `Height` are filled only while the drawing is open.
  Report "unknown", never zero.
- **`drawingObject.GetView()` throws.** That is why you read container by container and carry the
  view along yourself.
- **The type filter is narrower than the class tree.** `MarkBase`, `DimensionBase` and
  `DimensionSetBase` work; `SectionMarkBase` throws, so name the concrete classes. If a filter is
  rejected, enumerate everything and test types in C#.
- **Radius and curved dimensions expose points, not a value.**

## If you really need the drawing open

Opening is the one thing here that can cost the user work. Chapter 8 has the rule; in short:

1. Only for data that exists no other way (view frame sizes), only if the user asked, and only if
   **no** drawing is open: opening replaces the open one.
2. Check again immediately before the call. `SetActiveDrawing(drawing, false)` opens it hidden. It
   returns `false`, or throws, for a drawing that is locked, in use, or not up to date.
3. Read the views at once: view objects stay usable after closing.
4. Close with `CloseActiveDrawing(false)` (no save: you changed nothing), only after confirming the
   active drawing is still the one you opened. One attempt. Do not trust the return value: it has
   answered `false` for a drawing it did close **[verified 2026]**. Look at the editor again.

## Creating and editing drawings

- **Creating a GA drawing** **[verified 2026]**: construct `GADrawing` with a name and the name of a
  saved drawing-properties file (`"standard"` exists in the default environment), call `Insert()`.
  It does not open the editor. `drawing.Delete()` removes it again.
- Part-based drawings take the model object's identifier in the constructor; views come from the
  static `View.Create...` methods; texts, dimensions and marks are constructed with their view, then
  inserted **[lookup only]**. We have not verified whether such edits need the drawing open in the
  editor: assume they do **[inferred]**, under the rule above.
- After changing drawing objects call `drawing.CommitChanges()`: the drawing's own commit, called by
  your snippet like the model's, and not part of the model's undo step. The vendor documentation
  says it does not write to disk; saving is still the user's step **[lookup only]**.
- A snippet that touched only drawings has nothing for the model's commit to close, so it does not
  call it **[inferred]**.
- `IssueDrawing`, `UnissueDrawing`, `UpdateDrawing` and the settable status properties change a
  record other people rely on **[lookup only]**. Never without an explicit request.
