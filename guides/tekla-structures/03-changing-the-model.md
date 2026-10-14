# 3. Changing the model

Use this chapter when you create, modify or delete model objects: parts, bolts, welds, components,
attributes.

Essentials: lengths are mm; every change takes effect the moment you call it and **cannot be rolled
back by code**; nothing is committed for you, so the snippet calls the model's commit itself, once,
at the end; nothing is saved for you, and you must not save. Read chapter 8 first.

`model` in this chapter is a connected `Tekla.Structures.Model.Model`, if your Dynamic Bridge
profile defines an object under that name. The short type names assume the profile's usings.
`dotnet_list_profiles` shows what yours defines (chapter 0); a full type name always works.

## Before the first write

1. **Find the targets with a run that only reads.** Work from GUIDs you have just read, not from
   names or from a list made earlier in the conversation.
2. **Check the catalog** (chapter 6) for every section profile, material, bolt standard and
   component you will use. Names depend on the environment the model was made with: `HEA300` and
   `S355J2` exist in one office and not in the next.
3. **Tell the user what will change**: how many objects, which ones, which values.
4. **Plan one run.** One run that ends with one commit is one undo step for the user.

## Creating

Build the object in memory, set its properties, call `Insert()`, check the `bool`. Nothing reaches
the model before `Insert()`. A beam **[verified 2026]**: construct `Beam` with a start and an end
point, set `Name`, `Class` and the two catalog strings, insert.

```csharp
beam.Profile.ProfileString = "HEA300";     // a name the catalog search returned
beam.Material.MaterialString = "S355J2";   // likewise
```

- After a successful `Insert()` the object's `Identifier.GUID` is filled in **[verified 2026]**. Put
  it in the result at once: it is how you and the user find the object again.
- `Insert()` returning `false` is the only failure signal. No reason is attached.
- We have not recorded what the application does with a profile or material string it does not
  know. Do not find out on the user's model: that is what the catalog check is for.
- Where a part sits relative to its two points is set by `Position` (plane, depth, rotation and
  offsets) **[lookup only]**. Say which values you used, or copy them from a similar part.
- Mark what you create (a name prefix or a class the user agrees to), so it can be found again
  without the GUID list. It makes cleaning up after a failed run possible.

Plates, bolt groups, welds and connections follow the same pattern, with more inputs. We ran none of
them: look up the members and try on one object first **[lookup only]**.

## Modifying

Load the object by GUID (chapter 2), change properties on the loaded copy, call `Modify()`, check
the `bool`, read back and compare. Two things go wrong silently:

- **`Modify()` writes your whole copy, not only the field you touched** **[inferred]**. An old copy
  overwrites what the user changed in the meantime. Load, change and modify in the same run, close
  together.
- **Never modify an identity-only object.** One taken from an enumerator with
  `SelectInstances = false` has empty fields: `Select()` it first.

Attributes and placement have their own calls **[lookup only]**: `SetUserProperty(name, value)` for
UDAs (expect it to write at once, without `Modify()` **[inferred]**), `SetPhase(phase)`, and the
static `MoveObject`, `CopyObject`, `Split`, `Combine` on `Operation`.

## Deleting

Load by GUID, call `Delete()`, check the `bool`. Afterwards the GUID no longer resolves: the model
hands back an invalid identifier **[verified 2026]**. That is your proof.

- Delete only what the user named or what you created in this conversation. List it first, with
  counts by kind, and get a yes.
- Expect attached objects (cuts, bolts, welds) to go with it or be left broken **[inferred]**.

## The commit, and what cannot be previewed or undone

`Insert`, `Modify` and `Delete` are already in the model when they return. `model.CommitChanges()`
closes them into one undo step and redraws the views. The Dynamic Bridge commits nothing: **the
commit is yours to call**, as the snippet's last step, with a few words that say what the run did.

```csharp
bool committed = model.CommitChanges("Insert 12 beams, name prefix AA-");
```

- **Once per run**, after every `Insert`, `Modify` and `Delete`. Not after each object: every
  commit is a separate undo step for the user.
- The documentation marks the parameterless `CommitChanges()` as deprecated since version 2018; the
  overload with a message string carries no such mark **[lookup only]**. `false` from either means
  the step may not have been applied: read back before reporting.
- **A snippet that changed nothing in the model does not commit.** That covers a run that only
  reads, and equally one that only prints, exports, selects or runs the clash check.
- Drawings and catalogs do **not** ride on this commit. A drawing has its own
  `drawing.CommitChanges()`; a catalog item's `Insert()` or `Modify()` applies at once. Neither is
  part of the model's undo step.
- **There is no dry run.** A snippet that inserts and then "does not commit" has still inserted.
  The Guardrails setting Preview changes compiles and checks a snippet without running it: that
  proves the code compiles, and shows nothing of what it would change. To preview a change, compute
  it from what you read and describe it in words.
- **A run that fails half-way leaves the first half in the model**, and if it never reached the
  commit, that half is neither drawn nor in an undo step. Validate all inputs before the first
  `Insert()`. Collect the GUID of everything you create as you go and return the list even when the
  run fails. Put the commit where a failed run still reaches it (a `finally`, when at least one
  write succeeded), so that what is in the model is visible and is one step the user can undo
  **[inferred]**.
- **Uncommitted changes are not discarded.** Expect them to be drawn, and to join an undo step, at
  the next commit by anyone **[inferred]**.
- **Only the user can undo**, with the application's own undo, one step per commit. Undo does not
  reach files you wrote, drawings you changed, or catalog items.

## After the change

- Verify in a separate run that only reads, and that does not commit: the objects exist under the
  GUIDs you reported, the count moved by the number expected, the values are the ones asked for.
- Creating or changing parts makes numbering out of date **[inferred]**: check with
  `Operation.IsNumberingUpToDateAll()` (chapter 7) and tell the user. Do not run numbering, and do
  not update the drawings that show the changed parts: report both.
- In a shared or multi-user model (`model.GetInfo().SharedModel`, `SingleUserModel`) other people
  can be affected by what you write **[inferred]**: say so first. Remind the user that nothing is
  saved.
