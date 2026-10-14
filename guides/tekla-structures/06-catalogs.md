# 6. Catalogs

Use this chapter when you need a profile, material, bolt, rebar size, shape or component that really
exists in the user's Tekla Structures, before creating or changing anything, or to answer a question
about section properties.

Essentials: dimensions are mm (other units below); reading a catalog changes nothing and needs no
commit; **catalog contents depend on the environment the model was made with**, so never guess a
name.

"Profile" in this chapter is a steel or concrete section from the catalog. The other profile, the
Dynamic Bridge's own, decides how you reach the catalog namespace: through a using, through an
alias, or by full name. `dotnet_list_profiles` shows what yours defines (chapter 0).

## The way in

Create a `CatalogHandler` (namespace `Tekla.Structures.Catalogs`) with `new` and check
`GetConnectionStatus()`. Each catalog comes as an enumerator:

| Catalog | Call | Item class |
|---|---|---|
| Library profiles | `GetLibraryProfileItems()` | `LibraryProfileItem` |
| Parametric profiles | `GetParametricProfileItems()` | `ParametricProfileItem` |
| Materials | `GetMaterialItems()` | `MaterialItem` |
| Bolts | `GetBoltItems()` | `BoltItem` |
| Rebar | `GetRebarItems()` | `RebarItem` |
| Shapes (items) | `GetShapeItems()` | `ShapeItem` |
| Components | `GetComponentItems()` | `ComponentItem` |

A `null` enumerator is an error, not an empty catalog; the API can also throw
`CatalogItemEnumeratorInitializationException` when it cannot open a catalog at all.

There is no search call. Enumerate, match in C#, sort (digit runs as numbers: `HEA100` before
`HEA1000`), and only then read details. To test one name directly, `LibraryProfileItem.Select(name)`
and `MaterialItem.Select(name)` return `true` when it exists **[lookup only]**.

## Profiles: two catalogs, one string

A part's profile string comes from either catalog, and you must know which:

- A **library** profile is used by its name as it stands: `HEA300`.
- A **parametric** profile is a prefix you complete with dimensions: prefix `HI` plus values gives
  `HI300-15-20*300`. The item's `ParameterString` (for example `h-s-t*b`) gives the order of the
  values **and the separators**; `CreateProfileString()` returns a valid example built from the
  catalog's defaults. Follow the pattern exactly.

**Cost** **[verified 2026]**, default environment, 1,968 library and 285 parametric profiles: the
library enumerated without loading items takes about 0.2 s and gives name and type only; letting the
enumerator load every item takes about 8.5 s; the whole parametric catalog, loaded, about 0.2 s.

So: set `SelectInstances = false` on the library enumerator, match unloaded, then `Select()` only
the rows you return. A parametric item has **no type until it is loaded**, and that catalog is
small: load it all. An item whose `Select()` returns `false` is listed but not readable: say so, do
not return a thin row.

**Dimensions and section properties.** A loaded profile has `aProfileItemParameters` (dimensions),
and a library profile also `aProfileItemAnalysisParameters` (cross-section area, weight per length
and the like) and `aProfileItemUserParameters`. Each entry is a `ProfileItemParameter`: `Property`
(its name), `Symbol` (the letter in the section sketch and in a parametric pattern), a unit type,
and the value.

- **The value sits in one of three fields, chosen by the unit type**: `StringValue` for a string
  parameter, `IntegerValue` for integer and boolean ones, `Value` for everything else. The other two
  hold leftovers: `IntegerValue` of a decimal parameter is garbage.
- **Unset reads as 0.** The catalog declares some 25 section properties for every profile and fills
  in a handful. Leave zeros out; do not report them as values.
- For a rolled I section the first dimensions with a value are height, width, web thickness, flange
  thickness. A user-defined profile carries its template's whole list, mostly unset; sketched
  profiles name parameters by a label id (`albl_Width_of_outstand`).
- **Tapered and haunched profiles have several cross sections** (`NumberOfCrossSections` > 1). Every
  property then repeats once per section, told apart by `CrossSectionNumber`. Collected by name
  alone, the last section silently overwrites the first.

**Two traps in the type enums:** `PROFILE_L` and `PROFILE_FPL` are declared with the same number, so
an ordinary angle prints as `PROFILE_FPL`; and a user-defined profile's sub-type has no name, it
prints as digits.

## Materials, bolts, rebar, shapes, components

**Materials.** `MaterialItem`: `MaterialName`, `Type` (steel, concrete, timber, rebar,
miscellaneous), three alias names, `ProfileDensity`, `PlateDensity`, `ModulusOfElasticity`,
`PoissonsRatio`, `ThermalDilatation`, `DesignCode`. Search the aliases as well as the name.
**Strengths (yield, ultimate) are not available from the catalog API.** Do not derive them from the
name and present them as read from Tekla Structures.

**Bolts.** One entry per **standard and size**: `Standard`, `Type` (bolt or stud), `Size`,
`Lengths`. A bolt in the model is set by standard plus size (`BoltStandard`, `BoltSize`), so group
the entries by standard and list the sizes under each.

**Rebar.** `RebarItem`: `Grade`, `Size`, `Usage`, `Code`, `NominalDiameter`, `ActualDiameter`,
`CrossSectionArea`, bending and hook radii and lengths, `LapLength`; the weight property is spelled
`WeightPerLenght` in the API. A bar is identified by **grade + size + usage**: the same grade and
size appears twice, as a main bar and as a tie or stirrup, with different bending radii. Picking by
grade and size alone picks one of them at random.

**Shapes.** `ShapeItem`: `Name`, `Guid`, `IsSolid`, and `Extrema` (a bounding box, can be absent).

**Components.** `ComponentItem`: `Name`, `UIName`, `Number`, `Type`, `AttributeFileExtension`.
**System components have no API name, only a number**; `UIName` is what the user sees in the
Applications & components list and will ask for. Plugins and custom components are inserted by name.
Search `UIName` first, then `Name`, and return both with the number.

**Units and numbers.** Internal units throughout: mm, mm2, mm3, mm4, kg/m, kg/m3, N/m2. The catalogs
store single-precision values and convert units on the way out, so 16.7 arrives as 16.70000076 and
561000 as 560999.9895. Round to seven significant digits: that removes the noise without touching
real data.

## Calls that change a catalog

Reading is enumerating and `Select()`. Everything else writes, applies at once, and is not part of
the model's undo. Never call any of these unless the user asked for exactly that:

- `Insert()`, `Modify()`, `Delete()`, `Copy()` on an item; the `Modify...Parameter` and
  cross-section methods on a profile; `Export` on an item (writes a file).
- `Import...`, `Export...` and `SaveProfileDatabase()` on the handler.
- **`ProfileItem.GetProfileItemSubTypes` is a "Get" that writes**: it updates the sub-types in the
  profile database. Read the item's own properties instead.
