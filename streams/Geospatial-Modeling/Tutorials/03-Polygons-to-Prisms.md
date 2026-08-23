# Polygons to Solids: Extrusion, Triangulation, and Mesh Validity

*A hands-on guide to turning projected airspace footprints into closed 3D prisms that a containment test, a simulator, and a 3D printer can all trust.*

This tutorial assumes you have worked through the coordinate systems lesson. Every footprint here is already in meters before anything vertical happens to it.

## The problem in one sentence

An airspace feature is a 2D polygon plus a floor and a ceiling, and the extrusion step turns that into a solid; the difficulty is that most ways of doing this produce a mesh that looks right on screen and gives wrong answers to the only question we care about, whether a point is inside it.

---

## Table of Contents

1. [Why This Matters for the Airspace Project](#1-why-this-matters-for-the-airspace-project)
2. [Polygons Are Pickier Than They Look](#2-polygons-are-pickier-than-they-look)
3. [Triangulation: Why and How](#3-triangulation-why-and-how)
4. [Extruding the KLAF Class D](#4-extruding-the-klaf-class-d)
5. [Watertight, Manifold, and the Euler Number](#5-watertight-manifold-and-the-euler-number)
6. [Breaking the Mesh on Purpose](#6-breaking-the-mesh-on-purpose)
7. [Holes: Exclusion Cutouts and Torus Topology](#7-holes-exclusion-cutouts-and-torus-topology)
8. [Stacking Shelves: Class B and C Structures](#8-stacking-shelves-class-b-and-c-structures)
9. [Export: Simulator Meters and Printer Millimeters](#9-export-simulator-meters-and-printer-millimeters)
10. [Common Pitfalls](#10-common-pitfalls)
11. [Exercises](#11-exercises)
12. [Summary Table](#12-summary-table)
13. [Further Reading](#13-further-reading)

---

## 1. Why This Matters for the Airspace Project

Three consumers wait at the end of this pipeline, and each punishes a different defect. The geofence library needs a point-in-solid test, which silently returns wrong answers on meshes with missing faces or inconsistent winding. The simulator and viewer need meshes that load and shade correctly, which flipped normals ruin. The 3D printer needs a watertight boundary, and slicers reject or mangle anything else. None of these consumers warns you at build time; the defects surface later, as a geofence that leaks, a solid that renders inside-out, or a print that fails at 2 a.m.

The extrusion literature learned this lesson on city models. Ledoux and Meijers showed that extruding building footprints naively, without attending to the topology of the input polygons, yields models fit only for visualization, and their fix, a constrained triangulation of the footprint before extrusion, is the same procedure we use here. The validation side has an international standard, ISO 19107, and a free tool, val3dity, that checks solids against it. This tutorial builds the habits; the standard and the tool are in the reading section.

A note on the data used below. The airport CSV in our repository has no geometry column, so this tutorial reconstructs the KLAF Class D as a 4.4 NM circle around the airport reference point, which makes every example self-contained and reproducible. The published feature in `Class_Airspace.geojson` is close to circular but not exactly this, so when you move to the pipeline, load the real polygon and treat the circle as the sanity check. That substitution is itself an assumption log entry.

---

## 2. Polygons Are Pickier Than They Look

Before extruding anything, the footprint has to be a valid polygon, and validity means more than having vertices. A polygon is a closed ring of coordinates that does not cross itself, wound in a consistent direction, with interior rings (holes) wound opposite to the exterior. Software disagrees about some of this, which is exactly why it bites.

```python
from shapely.geometry import Polygon
from shapely.validation import explain_validity

# A square, its ring direction, and a bowtie that only looks like a polygon
square = Polygon([(0, 0), (10, 0), (10, 10), (0, 10)])
print(f"square valid: {square.is_valid},  exterior counterclockwise: {square.exterior.is_ccw}")

clockwise = Polygon([(0, 0), (0, 10), (10, 10), (10, 0)])
print(f"clockwise ring still 'valid' to shapely: {clockwise.is_valid},  ccw: {clockwise.exterior.is_ccw}")

bowtie = Polygon([(0, 0), (10, 10), (10, 0), (0, 10)])
print(f"bowtie valid: {bowtie.is_valid}")
print(f"reason: {explain_validity(bowtie)}")
print(f"bowtie .area still returns a number: {bowtie.area}")
```

**Verified output:**
```
square valid: True,  exterior counterclockwise: True
clockwise ring still 'valid' to shapely: True,  ccw: False
bowtie valid: False
reason: Self-intersection[5 5]
bowtie .area still returns a number: 0.0
```

Three lessons in eleven lines. First, shapely accepts clockwise exteriors as valid, but downstream consumers often do not: the GeoJSON specification wants counterclockwise exteriors, and triangulators and extruders use winding to decide which side is inside. Normalize orientation on the way in rather than hoping. Second, `explain_validity` tells you not only that a polygon is broken but where, and the coordinate it names is where you start debugging. Third, and most important as a habit: the bowtie is invalid, and shapely still computed an area for it. Its two lobes cancel to zero here, which is at least conspicuous; with unequal lobes you get a plausible-looking wrong number. Geometry libraries answer the question you asked, not the question you meant, so validate inputs explicitly with `is_valid` before any pipeline step, and fix rather than filter, since a dropped airspace feature is a hole in the geofence.

The FAA file arrives clean by GIS standards, but the moment your code buffers, simplifies, unions, or clips a polygon, you have manufactured a new one that has never been checked, and the check costs one line.

---

## 3. Triangulation: Why and How

Graphics hardware, mesh formats, and every geometry test downstream operate on triangles, so the polygon footprint must be triangulated before it can cap a solid. For a convex footprint any method works, including the naive fan from one vertex. Real airspace footprints are not convex: they have carve-outs, notches for satellite airports, and interior rings, and a fan across a concave region generates triangles outside the polygon.

Two families of algorithms handle the general case. Ear clipping walks the boundary slicing off one triangle at a time, is simple and fast, and handles holes; the `mapbox_earcut` library trimesh uses belongs to this family. Constrained Delaunay triangulation additionally optimizes triangle shape and guarantees the polygon's edges appear in the output, which is why Ledoux and Meijers build their extrusion on it; the classic implementation is Shewchuk's Triangle. For our purposes the practical guidance is short: let `trimesh.creation.extrude_polygon` call its triangulator, and check the result rather than the algorithm, because the validity tests in Section 5 catch a bad triangulation regardless of which method produced it.

---

## 4. Extruding the KLAF Class D

The full recipe, start to finish: build the footprint in degrees, project to a local metric frame, extrude between floor and ceiling, translate to true altitude, and interrogate the result. The vertical numbers come from the attribute table we read in the introduction tutorial: surface to 3,100 ft MSL, with field elevation about 606 ft, and the flat floor at field elevation is an approximation logged as such (the terrain drape that removes it is the next tutorial's subject).

```python
import numpy as np
import trimesh
from pyproj import Geod, Transformer
from shapely.geometry import Polygon
from shapely.ops import transform

geod = Geod(ellps="WGS84")

# KLAF Class D, reconstructed as a 4.4 NM circle around the airport reference point.
# Published vertical extent: SFC to 3,100 ft MSL. Field elevation is about 606 ft MSL.
lon0, lat0 = -86.9369, 40.4123
radius_m = 4.4 * 1852.0

az = np.linspace(0, 360, 361)
lons, lats, _ = geod.fwd(np.full_like(az, lon0), np.full_like(az, lat0),
                         az, np.full_like(az, radius_m))
footprint_deg = Polygon(zip(lons, lats))

# Tutorial 1: project to a local metric frame before doing any geometry
local = f"+proj=aeqd +lat_0={lat0} +lon_0={lon0} +datum=WGS84 +units=m"
tr = Transformer.from_crs("EPSG:4326", local, always_xy=True)
footprint_m = transform(tr.transform, footprint_deg)

# Vertical extent in meters: field elevation 606 ft up to ceiling 3,100 ft MSL
ft = 0.3048
floor_m, ceiling_m = 606 * ft, 3100 * ft
height = ceiling_m - floor_m

# Triangulate the footprint and pull it upward into a prism
prism = trimesh.creation.extrude_polygon(footprint_m, height=height)
prism.apply_translation([0, 0, floor_m])          # place the floor at its true altitude

print(f"vertices: {len(prism.vertices)}   faces: {len(prism.faces)}")
print(f"watertight:          {prism.is_watertight}")
print(f"winding consistent:  {prism.is_winding_consistent}")
print(f"is_volume:           {prism.is_volume}")
print(f"euler number:        {prism.euler_number}   (2 = topological sphere)")
print(f"z range:             {prism.bounds[0][2]:.1f} to {prism.bounds[1][2]:.1f} m MSL")
print(f"enclosed volume:     {prism.volume/1e9:.3f} km^3")
print(f"pi * r^2 * h check:  {np.pi * radius_m**2 * height / 1e9:.3f} km^3")
```

**Verified output:**
```
vertices: 720   faces: 1436
watertight:          True
winding consistent:  True
is_volume:           True
euler number:        2   (2 = topological sphere)
z range:             184.7 to 944.9 m MSL
enclosed volume:     158.572 km^3
pi * r^2 * h check:  158.580 km^3
```

The closing pair of lines is the pattern to copy everywhere: an independent arithmetic check against the mesh's own report. They differ by 0.005%, which is the polygon's 360 segments approximating a circle, and that residual is itself informative, since it shrinks as you densify and grows as you simplify. The z range confirms the translate step put the floor at 184.7 m, which is 606 ft, rather than leaving the solid sitting on zero, a mistake that costs exactly one airport elevation of geofence error and is invisible in a top-down view.

---

## 5. Watertight, Manifold, and the Euler Number

The three checks in that output are worth defining precisely, because they test different things and all three have to pass.

**Watertight** means every edge in the mesh is shared by exactly two triangles, so the surface has no boundary, no gaps a ray could slip through. **Winding consistent** means adjacent triangles agree about orientation, so all normals point outward and inside is well defined everywhere; trimesh's `is_volume` requires both plus positive volume. **The Euler number**, vertices minus edges plus faces, is a topological invariant: 2 for anything sphere-like, which a solid prism is, and 0 for anything with one tunnel through it, which Section 7 produces deliberately. It functions as a cheap fingerprint, since a prism reporting anything but 2 has a structural defect somewhere even if you have not found it yet.

The reason to internalize all three rather than just one is that they fail independently, which the next section demonstrates.

---

## 6. Breaking the Mesh on Purpose

Deliberately damaging a known-good mesh is the fastest way to learn what each check catches and, more usefully, what each check misses.

```python
import numpy as np
import trimesh
from shapely.geometry import Polygon

# Rebuild a small prism, then damage it two ways and watch the checks catch it
square = Polygon([(0, 0), (1000, 0), (1000, 1000), (0, 1000)])
mesh = trimesh.creation.extrude_polygon(square, height=500.0)
print(f"intact:        watertight={mesh.is_watertight}  is_volume={mesh.is_volume}  "
      f"euler={mesh.euler_number}  volume={mesh.volume/1e6:.1f}e6 m^3")

# Damage 1: delete one triangle. The mesh still renders; it just has a hole.
holed = mesh.copy()
holed.faces = holed.faces[:-1]
print(f"missing face:  watertight={holed.is_watertight}  is_volume={holed.is_volume}  "
      f"euler={holed.euler_number}  volume={holed.volume/1e6:.1f}e6 m^3  <- number no longer meaningful")

# Damage 2: flip the winding of one triangle. All faces exist; one normal points inward.
flipped = mesh.copy()
f = flipped.faces.copy()
f[0] = f[0][::-1]
flipped.faces = f
print(f"flipped face:  watertight={flipped.is_watertight}  is_volume={flipped.is_volume}  "
      f"winding_consistent={flipped.is_winding_consistent}")
```

**Verified output:**
```
intact:        watertight=True  is_volume=True  euler=2  volume=500.0e6 m^3
missing face:  watertight=False  is_volume=False  euler=1  volume=500.0e6 m^3  <- number no longer meaningful
flipped face:  watertight=True  is_volume=False  winding_consistent=False
```

Read the last line carefully, because it carries the lesson of the whole section: the mesh with a flipped triangle is still watertight. Every edge still touches two faces, so the watertightness test passes while the solid is unusable, and only the winding check catches it. And in the middle line, note that the holed mesh still reports a volume, one that happens to round to the correct value here, which is worse than an error because nothing about the number announces that the surface it summarizes no longer encloses anything. The rule that falls out of this: a mesh enters the pipeline only after `is_volume` passes, and a mesh that fails is repaired and rechecked, never waved through because its numbers look reasonable.

Both damaged meshes render normally in a viewer, which is why on-screen inspection certifies nothing.

---

## 7. Holes: Exclusion Cutouts and Torus Topology

Some airspace features carry the `EXCLUSION` flag, meaning a region cut out of the interior, and geometrically that is a polygon with an interior ring. Extrusion handles it, but the result is a different kind of object, and the checks reflect that.

```python
import numpy as np
import trimesh
from shapely.geometry import Polygon

# An exclusion cutout: outer ring plus one interior ring, extruded together.
outer = [(0, 0), (10000, 0), (10000, 10000), (0, 10000)]
inner = [(4000, 4000), (4000, 6000), (6000, 6000), (6000, 4000)]   # note: clockwise
donut = Polygon(outer, holes=[inner])
print(f"polygon valid: {donut.is_valid}   area: {donut.area/1e6:.0f} km^2 "
      f"(100 outer - 4 hole = 96)")

prism = trimesh.creation.extrude_polygon(donut, height=1000.0)
print(f"watertight: {prism.is_watertight}   is_volume: {prism.is_volume}")
print(f"euler number: {prism.euler_number}   (0 = torus: one tunnel through the solid)")
print(f"volume: {prism.volume/1e9:.1f} km^3   (96 km^2 * 1 km = 96)")

# Containment respects the hole
inside_solid  = trimesh.proximity.signed_distance(prism, [[2000, 2000, 500]])[0]
inside_cutout = trimesh.proximity.signed_distance(prism, [[5000, 5000, 500]])[0]
print(f"signed distance, point in the solid:  {inside_solid:+.0f} m  (positive = inside)")
print(f"signed distance, point in the cutout: {inside_cutout:+.0f} m  (negative = outside)")
```

**Verified output:**
```
polygon valid: True   area: 96 km^2 (100 outer - 4 hole = 96)
watertight: True   is_volume: True
euler number: 0   (0 = torus: one tunnel through the solid)
volume: 96.0 km^3   (96 km^2 * 1 km = 96)
signed distance, point in the solid:  +500 m  (positive = inside)
signed distance, point in the cutout: -1000 m  (negative = outside)
```

The Euler number dropped from 2 to 0 because a vertical tunnel now passes through the solid, and that is correct, not a defect: the fingerprint changed because the topology changed. This is why the Section 5 rule says a prism should report 2, phrased with the topology in mind; the honest general rule is that the Euler number should match the number of holes you know the footprint has, 2 minus 2 per tunnel. The signed distance calls at the end are the payoff and a preview of the containment tutorial: an aircraft over the excluded region at 500 m is reported outside the airspace, at a distance of 1,000 m from the nearest wall, which is the hole's half-width. The geometry, the arithmetic, and the operational meaning all agree, and that three-way agreement is what this pipeline exists to deliver.

One caution on hole winding: the interior ring above was given clockwise, opposite to the exterior, which is the convention. Some triangulators tolerate holes wound the wrong way and some produce garbage, so normalize hole orientation with the same discipline as exteriors.

---

## 8. Stacking Shelves: Class B and C Structures

A Class B or C is not one solid; it is a stack of them, an inner core from the surface and outer shelves with raised floors, published as separate features sharing an `ADHP_ID`. The pipeline groups by that key and builds each shelf as its own prism, and the containment question becomes which member of the family, if any, contains the point. A toy two-shelf version, with round numbers standing in for any particular airport's:

```python
import numpy as np
import trimesh
from shapely.geometry import Point

ft = 0.3048

# A toy Class C wedding cake: inner cylinder SFC to 4,600 ft AGL-ish values in MSL,
# outer shelf from 1,900 ft up to the same ceiling. (Real numbers vary per airport.)
core_footprint  = Point(0, 0).buffer(5 * 1852.0, quad_segs=90)     # 5 NM radius
shelf_footprint = Point(0, 0).buffer(10 * 1852.0, quad_segs=90).difference(core_footprint)

core  = trimesh.creation.extrude_polygon(core_footprint,  height=(4600 - 600) * ft)
core.apply_translation([0, 0, 600 * ft])
shelf = trimesh.creation.extrude_polygon(shelf_footprint, height=(4600 - 1900) * ft)
shelf.apply_translation([0, 0, 1900 * ft])

for name, m in [("core", core), ("shelf", shelf)]:
    print(f"{name:6s} watertight={m.is_watertight}  euler={m.euler_number}  "
          f"volume={m.volume/1e9:7.2f} km^3")

# Sample points and ask which solid claims them: this is the containment query
tests = {"under the shelf, 1000 ft":  ( 12000, 0, 1000 * ft),
         "under the shelf, 3000 ft":  ( 12000, 0, 3000 * ft),
         "in the core,     1000 ft":  (  2000, 0, 1000 * ft),
         "outside both,    3000 ft":  ( 25000, 0, 3000 * ft)}
for label, p in tests.items():
    hit = "core" if core.contains([p])[0] else "shelf" if shelf.contains([p])[0] else "neither"
    print(f"{label}  ->  {hit}")
```

**Verified output:**
```
core   watertight=True  euler=2  volume= 328.42 km^3
shelf  watertight=True  euler=0  volume= 665.04 km^3
under the shelf, 1000 ft  ->  neither
under the shelf, 3000 ft  ->  shelf
in the core,     1000 ft  ->  core
outside both,    3000 ft  ->  neither
```

The four test points trace the wedding cake in words: low under the shelf is uncontrolled airspace, higher at the same spot is inside the shelf, low near the field is inside the core. That "neither" on the first line is the operationally famous gap that lets aircraft fly beneath a shelf without a clearance, and the fact that our geometry reproduces it is the kind of check no unit test framework hands you; it comes from knowing what the answer has to be. Note also the shelf's Euler number of 0, an annulus prism, a torus by construction rather than by exclusion flag, which is a second reminder that 0 is not an error code.

Two decisions this toy postpones for the real pipeline. Adjacent shelves share a wall, and building each independently duplicates geometry along it; the ISO 19107 CompositeSolid concept, and val3dity's checks for it, exist precisely for stacks like this, and whether we keep independent solids or a shared-face composite is an assumption log entry with performance on one side and validation strictness on the other. And the family's identity lives in the attributes, so the grouping join must survive the identifier pitfalls from the introduction tutorial.

---

## 9. Export: Simulator Meters and Printer Millimeters

STL, the exchange format for everything downstream, stores no units, only numbers, and every consumer imposes its own convention: simulators read our meters as meters because we tell them to, while slicers assume millimeters. So the same solid exports twice, once untouched for the geofence library and simulator, and once scaled for the printer, and the outreach model needs one more decision, vertical exaggeration, because airspace is wide and thin.

```python
# Rebuild the KLAF prism from Section 4, then export it two ways.
# Export 1: meters, for the simulator and the containment library
prism.export("klaf_class_d_meters.stl")

# Export 2: scaled for a desktop 3D print. STL has no units; slicers assume mm.
# Horizontal 1:100,000 and vertical exaggerated 10x so the ceiling is visible.
model = prism.copy()
model.apply_scale([1000 / 100000, 1000 / 100000, 10 * 1000 / 100000])
model.export("klaf_class_d_print.stl")

d_mm = model.bounds[1][0] - model.bounds[0][0]
h_mm = model.bounds[1][2] - model.bounds[0][2]
print(f"print model: diameter {d_mm:.0f} mm, wall height {h_mm:.1f} mm, "
      f"watertight={model.is_watertight}")
print("without the 10x vertical exaggeration the wall would be "
      f"{h_mm/10:.2f} mm tall")
```

**Verified output:**
```
print model: diameter 163 mm, wall height 76.0 mm, watertight=True
without the 10x vertical exaggeration the wall would be 7.60 mm tall
```

At true proportions the entire Class D, surface to 3,100 ft, is a 7.6 mm lip on a 163 mm disk, which is a physical fact about airspace worth a moment: the volumes we draw as towering wedding cakes are, at scale, coats of paint. The 10x exaggeration that makes the model legible is a distortion introduced on purpose, so it goes on the model's label and in the assumption log, the same way a subway map admits it is not to scale. Rerun the validity checks after any scale operation, as the code does; anisotropic scaling is exactly the kind of transform that turns marginal geometry into broken geometry.

---

## 10. Common Pitfalls

1. **Extruding an unvalidated polygon.** Buffers, unions, clips, and simplification all mint new polygons. One `is_valid` call per minting, and `explain_validity` when it fails.
2. **Trusting the picture.** Every damaged mesh in Section 6 renders normally. Certification comes from `is_volume`, not from looking.
3. **Treating watertight as the whole test.** The flipped-face mesh passes watertightness and fails winding. Check both, and let the Euler number confirm the topology you expect.
4. **Forgetting the floor translation.** `extrude_polygon` builds from z equals 0; the translate to floor altitude is a separate step, and omitting it produces a geofence one field-elevation too low that no top-down view will ever reveal.
5. **Mixing feet and meters mid-pipeline.** The FAA publishes vertical values in feet, the projected frame is meters, and the `ft = 0.3048` constant should appear exactly once, at the boundary.
6. **Assuming hole winding.** Interior rings wound the same direction as the exterior are accepted by some tools and mis-triangulated by others. Normalize, do not hope.
7. **Scaling without rechecking.** Export-time scale factors, especially anisotropic ones for print, warrant one more `is_watertight` before the file leaves the pipeline.
8. **Building shelves without their family.** A Class B core extruded alone is a valid solid and a wrong model. The grouping by `ADHP_ID` is part of the geometry, not a display concern.

---

## 11. Exercises

1. Load the real KLAF Class D polygon from `Class_Airspace.geojson`, run it through the Section 4 recipe, and compare its volume and footprint area against the 4.4 NM circle used here. How far off was the circle assumption, and would the difference matter to a geofence with a 100 m buffer?
2. Take the Section 4 prism and simplify its footprint with `simplify()` at 100 m tolerance before extruding. Report vertex count, volume change, and whether the result still passes `is_volume`. Then find a tolerance at which it stops passing and explain what broke, using `explain_validity` on the simplified footprint.
3. Build the Section 7 donut with the interior ring wound counterclockwise, same direction as the exterior, and report what your triangulator does with it. Compare against the correctly wound version by volume and Euler number.
4. Extend the Section 8 toy to three shelves and write a function `which_shelf(point)` returning the containing feature or None. Then place a test point exactly on the boundary between core and shelf and report what happens, in writing, because the answer previews the floating point discussion in the containment tutorial.
5. Export the Section 8 stack as a single STL containing both solids and run it through val3dity's web interface (geovalidation.bk.tudelft.nl). Report which ISO 19107 errors, if any, it raises for the shared wall, and reconcile the answer with trimesh's checks.
6. Print the KLAF model, or slice it and report the slicer's verdict, then compute the scale and exaggeration needed for a version of the Indianapolis Class B that fits a 200 mm build plate with its tallest shelf at least 40 mm.

---

## 12. Summary Table

| Question | Answer for this project |
|---|---|
| What comes in? | A valid, metrically projected, correctly wound footprint polygon |
| Who triangulates? | `trimesh.creation.extrude_polygon` and its earcut backend |
| What comes out? | A prism translated to its true floor altitude, in meters MSL |
| The acceptance test? | `is_volume`, which requires watertight plus consistent winding |
| The topology fingerprint? | Euler number: 2 per solid, minus 2 per tunnel through it |
| The arithmetic check? | An independent volume estimate agreeing to a stated tolerance |
| Exclusion cutouts? | Interior rings, wound opposite the exterior, Euler number 0 |
| Class B and C? | One prism per shelf, grouped by `ADHP_ID`, queried as a family |
| Export units? | Meters untouched for simulation; scaled to mm, and rechecked, for print |
| What renders proves? | Nothing |

---

## 13. Further Reading

Ledoux, H. and Meijers, M. (2011), *Topologically consistent 3D city models obtained by extrusion*, International Journal of Geographical Information Science 25(4), 557-574, DOI 10.1080/13658811003623277, free via the TU Delft repository, is the paper this tutorial's procedure descends from, and its treatment of shared walls between adjacent extrusions is the background for the Class B composite question. Ledoux, H. (2018), *val3dity: validation of 3D GIS primitives according to the international standards*, Open Geospatial Data, Software and Standards 3(1), DOI 10.1186/s40965-018-0043-x, open access, defines the ISO 19107 checks and documents the tool exercise 5 uses.

The trimesh documentation (trimsh.org) is short and worth reading end to end, particularly the sections on watertightness and repair; `trimesh.repair` contains the fix-up functions (normals, winding, hole filling) this tutorial deliberately did not reach for, because repair without understanding is how defects hide. For triangulation, Shewchuk's Triangle page documents constrained Delaunay triangulation with the clearest figures anywhere, and for mesh processing as a field, Botsch et al., *Polygon Mesh Processing*, is the standard text.

Next tutorial in this series: **Altitude Is Not One Thing: Vertical Datums and the Sentinel Problem**, where the flat floor assumed at 606 ft in Section 4 gets replaced with the terrain it was standing in for.
