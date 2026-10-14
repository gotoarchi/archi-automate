# 7. Selection, numbering and the clash check

Use this chapter when the user points at something instead of naming it ("these beams"), when you
want to show objects to the user, when marks on drawings or reports must be trusted, or when you run
the application's own clash check.

Essentials: objects are identified by GUID; lengths are mm; none of this changes the model, so none
of it calls the model's commit; but **the selection belongs to the user**: if you replace it, put
it back.

`model` in this chapter is a connected `Tekla.Structures.Model.Model`, if your Dynamic Bridge
profile defines an object under that name. The short type names assume the profile's usings.
`dotnet_list_profiles` shows what yours defines (chapter 0); a full type name always works.

## Selection

The class is `Tekla.Structures.Model.UI.ModelObjectSelector`, created with `new`. Write the full
name, or your profile's alias for that namespace: it is not the database selector you get from
`model.GetModelObjectSelector()`.

**Reading.** `GetSelectedObjects()` returns an enumerator. `GetSize()` is the count: use it instead
of walking a selection of thousands. Each step loads one object (a round trip), so stop at your
limit. With `SelectInstances = false` the walk is cheap and the objects are identity-only (a session
number, an empty GUID, empty fields), which is all a snapshot needs. Zero selected means: ask the
user to select the objects in Tekla Structures.

**Setting.** `Select(list, false)` takes an `ArrayList` of model objects and **replaces** the
selection. An empty list clears it.

1. Resolve the GUIDs to objects first (chapter 2); identity-only objects are enough. If ids were
   given and **none** resolved, do not call `Select`: an empty list would wipe the user's selection
   because of your mistake.
2. Take a snapshot of the current selection if you will need to restore it.
3. Call `Select`. `false` means the application refused: a command is running or a dialog is open,
   and the user must finish or cancel it (Esc).
4. **Read the selection back.** An object that exists but cannot be selected now (hidden, or
   filtered out of the view) is left out without any error.
5. Compare by **identity**, not by count: after a refused call the old selection can hold exactly as
   many objects as the one you wanted. Use `Identifier.ID`, which every reference carries, and fall
   back to the GUID.

Selecting shows a result to the user and sets the scope for the "from selected" exports and reports
(chapter 5). When it was only a means, restore the snapshot and verify by identity. If that fails,
tell the user: they will find other objects selected than the ones they chose.

## Numbering

Numbering gives parts and assemblies their position marks (`P/12`, `A/3`). Marks on drawings and in
reports are only right when it is up to date. Both checks are static methods on
`Tekla.Structures.Model.Operations.Operation`.

- `IsNumberingUpToDateAll()`: one fast call for the whole model, `true` only if every part, assembly
  and rebar is numbered and current. Ask it first; if `true`, there is nothing to list.
- `IsNumberingUpToDate(obj)`: one object; works on an identity-only object. It **throws
  `ArgumentException`** for a kind that is not numbered (a bolt, a weld, a grid): catch it and
  report "not a numbered kind". Numbered kinds are parts, assemblies, cast units, rebars, pours and
  surface treatments.
- To find the stale ones, enumerate `Part` and `Assembly` with `SelectInstances = false` and ask per
  object: tens of thousands fit in one run. Skipping ahead in such an enumerator is free, so a long
  scan can be cut into runs by offset while nobody edits the model.
- The mark itself is a report property (`PART_POS`, `ASSEMBLY_POS`, `REBAR_POS`), which can be asked
  of an identity-only object. A mark that is not current has `(?)` appended; `P/0(?)` means never
  numbered. Pass the text on unchanged.

**Never run numbering.** It can change marks that drawings and reports already carry **[inferred]**,
and it is the user's decision. The public `Operation` class has no method that runs it
**[lookup only]**; the only route would be a macro, which is out of bounds (chapter 8). Tell the
user it is out of date and let them run it.

## The clash check

The clash check of Tekla Structures is **event-driven and works on the selection**. No call takes a
list of objects and returns a list of clashes: you build that yourself, in this order. Size the set
first: 2 to a few hundred objects (a few hundred take under a second; the call accepts no deadline,
so the size of the set is your only control over the time). For more, or for the whole model,
export IFC and check the file with an IFC tool. The check uses the model's clash settings.

1. Create a `Tekla.Structures.Model.Events` object. Put each handler into a **variable of its
   delegate type**, attach the two variables to `ClashDetected` and `ClashCheckDone`, call
   `Register()`.
2. Snapshot the user's selection (identity-only).
3. Select the objects to check. Read the selection back and compare by identity: hidden or filtered
   objects are dropped silently, and the check would run on fewer objects than asked. Fewer than two
   selected: stop.
4. Call `model.GetClashCheckHandler().RunClashCheck()`. `false` means it did not start: another
   check is running (the user's Clash Check Manager) or a command is active.
5. Wait, with a time limit, for `ClashCheckDone`. It carries the application's own count of clashes.
   Then give stragglers a few seconds, until as many `ClashDetected` events have arrived as it says.
6. Detach both handlers with `-=` and the same two variables, then call `events.UnRegister()`.
7. Restore the user's selection, in a `finally`, and verify it.
8. Only now read the clashes.

```csharp
Events.ClashDetectedDelegate onClash = data => { lock (gate) received.Add(data); };
Events.ClashCheckDoneDelegate onDone = count => { lock (gate) reported = count; done.Set(); };
events.ClashDetected += onClash; events.ClashCheckDone += onDone; events.Register();
```

- **The delegate types** are `Events.ClashDetectedDelegate` (it receives a `ClashCheckData`) and
  `Events.ClashCheckDoneDelegate` (it receives the count as an `int`), both nested in
  `Tekla.Structures.Model.Events` **[lookup only]**.
- **A lambda written straight after `+=` cannot be detached.** A second lambda with the same text
  is a different delegate, so `-=` with it removes nothing and raises no error. That is what the
  two variables are for.
- **The `Events` object is released by `UnRegister()`**, spelled with a capital R. Do not count on
  disposing it instead.
- The drawing namespace has an `Events` class too **[lookup only]**. With the drawing namespace
  behind an alias, `Events` written short is the model's; when unsure, write the full name.
- A wait handle such as `ManualResetEvent` lives in `System.Threading`, which a snippet has only by
  full name unless your profile adds the using.

**Inside the handlers, store the argument and nothing else.** The events arrive on thread-pool
threads, several at once, without needing a message loop **[verified 2026]**. Even reading
`data.Object1` is an API call, and the API belongs to your snippet's thread.

**Reading a clash** (`ClashCheckData`), back on your own thread:

- `Object1`, `Object2`: each read is a lookup in the model, so read each once and keep it. Either
  can be `null` or have no GUID you can get: the clash is then "unidentified".
- **The two objects carry identity only** **[verified 2026]**: the class and the identifier, with
  name, profile and material empty. For those, take the GUID (ask `model.GetGUIDByIdentifier` when
  the identifier's own is empty) and load the object by GUID (chapter 2); for many clashes, load
  them all with one `model.FetchModelObjects` call.
- `Type`: `CLASH`, `ISINSIDE`, `EXACTMATCH` (duplicates), `CUTTHROUGH`, `MINDISTANCE`,
  `COMPLEXCLASH`. `FAILEDSOLID`, `FAILEDTEST` and `INVALID` mean the pair **could not be tested**:
  count those apart, they are not findings.
- `Overlap`: comes back as 0 for plain clashes of type `CLASH` **[verified 2026]**, although the
  documentation ties the value to that type **[lookup only]**. Zero means "not given", never "no
  overlap". Do not report it as a measurement.
- **There is no clash volume.** `ClashCheckData` has the two objects, the type and the overlap, and
  nothing else **[lookup only]**.
- Where: `GetIntersectionBoundingBoxes(idA, idB)` on the clash check handler, with identifiers built
  from the two objects' numeric `ID`, returns boxes in model coordinates; take their union. The
  boxes give an **approximate extent** only: the volume of a box is not the volume of the clash. One
  more round trip per clash, so do it for a limited number.

**Is the list the whole result?** Nothing in an event says which check raised it. The list is
complete only if the number of events received equals the count from `ClashCheckDone`, every event
could be read and names two identified objects, and every object asked for was really selected.
Otherwise say it is incomplete and why. More events than the count means another check ran at the
same time and its clashes are mixed in. Zero clashes with a complete result is a valid, clean
answer; never tell the user "no clashes" from an incomplete one.

**After a timeout** the check may still be running inside the application. `StopClashCheck()` asks
it to stop, but works only when the advanced option `XS_USE_NEW_CLASH_CHECK` is set. Do not start
another check until the old one's `ClashCheckDone` has arrived, or about a minute has passed without
any event: the two checks' clashes could not be told apart.

**Side effects**, though the model is unchanged: the selection is replaced for the length of the
check, and every check is appended to `ClashCheck.history` in the model folder. A handler left
registered keeps running after your snippet returns: always unregister. **A run that only ran the
clash check does not call the model's commit**: it changed nothing that a commit would close.
