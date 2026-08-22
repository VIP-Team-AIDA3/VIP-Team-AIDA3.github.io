# From Degrees to Meters: Coordinate Reference Systems and Projection Error

*A hands-on guide to why latitude and longitude are not distances, and what to do about it before extruding airspace polygons into 3D solids.*

You don't need any GIS background for this. If you can picture a globe and a flat map, that's enough.

## The problem in one sentence

Every polygon in `Class_Airspace.geojson` stores its vertices in degrees, and every question we want to ask of it, how large it is, how far the aircraft is from its boundary, what its extruded volume looks like, needs an answer in meters. The step between the two is a projection, and choosing it carelessly produces geometry that is quietly wrong everywhere. This tutorial is about making that step deliberately.

---

## Table of Contents

1. [Why This Matters for the Airspace Project](#1-why-this-matters-for-the-airspace-project)
2. [A Degree Is Not a Distance](#2-a-degree-is-not-a-distance)
3. [The Square-Degrees Trap, Live in Our Own Data](#3-the-square-degrees-trap-live-in-our-own-data)
4. [Choosing a Projection: Four Candidates Measured](#4-choosing-a-projection-four-candidates-measured)
5. [Straight Lines That Aren't: Edge Deviation](#5-straight-lines-that-arent-edge-deviation)
6. [The Axis-Order Trap](#6-the-axis-order-trap)
7. [Applying This to the Real File](#7-applying-this-to-the-real-file)
8. [Common Pitfalls](#8-common-pitfalls)
9. [Exercises](#9-exercises)
10. [Summary Table](#10-summary-table)
11. [Further Reading](#11-further-reading)

---

## 1. Why This Matters for the Airspace Project

The extrusion pipeline you are about to build takes a 2D polygon and pulls it upward between a floor and a ceiling. If the footprint is still in degrees when you do that, the resulting solid has three axes in three different units: the east axis in longitude degrees, the north axis in latitude degrees, and the vertical axis in feet. At Atlanta's latitude a degree of longitude is about 17% shorter than a degree of latitude, so the "circle" of the Mode C veil becomes an ellipse, every distance query returns a number with no unit, and the STL you print is stretched. Nothing errors out. The code runs, the viewer renders, and everything downstream is wrong by an amount that varies with latitude.

The fix is to project into a coordinate system whose units are meters before doing any geometry. Which projection, and how much error each choice costs, is the subject of this tutorial. The numbers below come from the same KATL Mode C feature that sits in row one of our airspace file, so you can rerun everything against the real data.

There is a certification-adjacent reason to care too. A geofence decision for Earhart is a containment test, and NASA's analysis of exactly this question (Neeley and Narkawicz, 2017) found that the answer to "is the aircraft inside the polygon" can flip depending on how the polygon's edges are drawn between its published vertices. Their recommended constraints, no vertex more than 100 NM from the projection point and no edge longer than 15 NM, come straight out of the kind of measurement we make in Section 5.

---

## 2. A Degree Is Not a Distance

Latitude lines are (almost) evenly spaced, so a degree of latitude is a nearly constant 111 km anywhere on Earth. Longitude lines converge at the poles, so a degree of longitude shrinks as you go north. `pyproj.Geod` computes true distances on the WGS84 ellipsoid, which makes it our ruler for everything that follows.

```python
from pyproj import Geod
geod = Geod(ellps="WGS84")

# How long is one degree of longitude? Depends where you are.
for name, lat in [("Equator", 0.0), ("KATL (Atlanta)", 33.63),
                  ("KLAF (Purdue)", 40.41), ("Deadhorse, AK", 70.19)]:
    _, _, dist = geod.inv(0.0, lat, 1.0, lat)
    print(f"{name:16s} 1 deg longitude = {dist/1000:8.3f} km")

# One degree of latitude, for comparison
_, _, dlat = geod.inv(-86.94, 40.0, -86.94, 41.0)
print(f"{'Anywhere':16s} 1 deg latitude  = {dlat/1000:8.3f} km  (nearly constant)")
```

**Verified output:**
```
Equator          1 deg longitude =  111.319 km
KATL (Atlanta)   1 deg longitude =   92.783 km
KLAF (Purdue)    1 deg longitude =   84.880 km
Deadhorse, AK    1 deg longitude =   37.838 km
Anywhere         1 deg latitude  =  111.044 km  (nearly constant)
```

Read the Deadhorse row twice. Our airspace file covers Alaska, where a degree of longitude is barely a third of a degree of latitude. Any code that treats the two interchangeably is off by a factor of three there, not a rounding error. At KLAF the ratio is about 0.76, which is small enough to hide during a demo and large enough to matter in a geofence.

---

## 3. The Square-Degrees Trap, Live in Our Own Data

Open the KATL Mode C record in the airspace file and look at its `Shape__Area` attribute: 0.9424. Square what, exactly? Let's find out by rebuilding the veil ourselves. The Mode C veil is defined as a 30 nautical mile circle around the airport reference point, so we can construct it as a geodesic circle and compare.

```python
import numpy as np
from pyproj import Geod
from shapely.geometry import Polygon

geod = Geod(ellps="WGS84")

# Reconstruct the KATL Mode C veil: a 30 NM circle around the Atlanta ARP
lon0, lat0 = -84.4277, 33.6367          # KATL airport reference point
radius_m = 30 * 1852.0                  # 30 nautical miles in meters

az = np.linspace(0, 360, 721)
lons, lats, _ = geod.fwd(np.full_like(az, lon0), np.full_like(az, lat0),
                         az, np.full_like(az, radius_m))
veil = Polygon(zip(lons, lats))

# The trap: shapely computes area in the polygon's own units. Degrees in, square degrees out.
print(f"shapely veil.area                  = {veil.area:.4f}   <- square DEGREES")
print(f"FAA attribute Shape__Area for KATL = 0.9424   <- same meaningless unit")

# Ground truth: geodesic area on the WGS84 ellipsoid
area_m2, perim_m = geod.geometry_area_perimeter(veil)
print(f"geodesic area  = {abs(area_m2)/1e6:10.1f} km^2")
print(f"geodesic perim = {perim_m/1000:10.1f} km")
print(f"pi * (55.56 km)^2 sanity check = {np.pi * 55.56**2:8.1f} km^2")
```

**Verified output:**
```
shapely veil.area                  = 0.9424   <- square DEGREES
FAA attribute Shape__Area for KATL = 0.9424   <- same meaningless unit
geodesic area  =     9697.6 km^2
geodesic perim =      349.1 km
pi * (55.56 km)^2 sanity check =   9697.8 km^2
```

Three things just happened. First, our reconstructed circle's raw shapely area matches the FAA's published `Shape__Area` to four decimal places, which confirms both that the veil really is a 30 NM circle and that the FAA attribute is in square degrees, a unit with no physical meaning. Second, the geodesic area agrees with plain circle arithmetic to within rounding, which is the sanity check you should always run when a new tool enters the pipeline. Third, and this is the habit to take away: shapely is a flat-plane library. It does exactly what you ask in whatever units you feed it, and it will never warn you that degrees were a bad idea.

The rule for the whole project: **shapely does geometry, pyproj does the Earth.** Anything still in degrees may be stored and displayed, never measured.

---

## 4. Choosing a Projection: Four Candidates Measured

So we project. Into what? A projection flattens part of the curved Earth onto a plane, and every projection distorts something. The question is never "which projection is correct" but "how much error does each one cost for this region and this purpose." So we measure, using the geodesic area from Section 3 as ground truth.

```python
import numpy as np
from pyproj import Geod, Transformer
from shapely.geometry import Polygon
from shapely.ops import transform

geod = Geod(ellps="WGS84")
lon0, lat0 = -84.4277, 33.6367
az = np.linspace(0, 360, 721)
lons, lats, _ = geod.fwd(np.full_like(az, lon0), np.full_like(az, lat0),
                         az, np.full(az.shape, 30 * 1852.0))
veil = Polygon(zip(lons, lats))
truth_km2 = abs(geod.geometry_area_perimeter(veil)[0]) / 1e6

def projected_area(epsg_or_proj):
    tr = Transformer.from_crs("EPSG:4326", epsg_or_proj, always_xy=True)
    return transform(tr.transform, veil).area / 1e6

candidates = {
    "Local azimuthal equidistant (centered on KATL)":
        f"+proj=aeqd +lat_0={lat0} +lon_0={lon0} +datum=WGS84 +units=m",
    "UTM zone 16N (EPSG:32616)": "EPSG:32616",
    "UTM zone 17N (EPSG:32617)  <- wrong zone for the ARP": "EPSG:32617",
    "Web Mercator (EPSG:3857)   <- never for measurement": "EPSG:3857",
}

print(f"{'projection':55s} {'area km^2':>10s} {'error':>8s}")
print(f"{'geodesic (ground truth)':55s} {truth_km2:10.1f} {'-':>8s}")
for name, crs in candidates.items():
    a = projected_area(crs)
    print(f"{name:55s} {a:10.1f} {100*(a-truth_km2)/truth_km2:+7.2f}%")
```

**Verified output:**
```
projection                                               area km^2    error
geodesic (ground truth)                                     9697.6        -
Local azimuthal equidistant (centered on KATL)              9697.7   +0.00%
UTM zone 16N (EPSG:32616)                                   9703.7   +0.06%
UTM zone 17N (EPSG:32617)  <- wrong zone for the ARP        9714.3   +0.17%
Web Mercator (EPSG:3857)   <- never for measurement        14027.3  +44.65%
```

How to read this table:

**The local azimuthal equidistant projection wins for our purpose.** Centered on the airport we are modeling, it preserves distance and direction from that center essentially perfectly at this scale. This is the projection the extrusion pipeline should use, one instance per airport, recentered each time. It is the coordinate-system equivalent of an ENU frame in GNC, and the analogy is exact: a locally flat plane tangent to the Earth at your point of interest.

**UTM is the respectable general-purpose answer** and costs 0.06% here. Notice something interesting though: KATL sits at 84.43°W, and 84°W is precisely the boundary between UTM zones 16 and 17. The veil straddles the line, so neither zone covers it without some out-of-zone distortion, and the difference between choosing zone 16 and zone 17 shows up in the third row. When a feature crosses a zone boundary, pick the zone containing its centroid and note the choice in your assumption log, or use a local projection and sidestep the issue.

**Web Mercator is 44.65% wrong and this is not a bug.** EPSG:3857 is what every web basemap uses, so its coordinates are lying around everywhere and it is the single easiest projection to reach for by accident. Its area distortion grows as 1/cos²(latitude), which at 33.6°N is a factor of 1.44, exactly what we measured. It exists to make map tiles align, not to measure anything. If a distance or area in your pipeline came out of EPSG:3857, it is wrong, and at KLAF's latitude it is wrong by about 72%.

---

## 5. Straight Lines That Aren't: Edge Deviation

The airspace file gives us vertices. The edges between them have to be drawn by something, and there are at least two reasonable choices: a straight line in latitude-longitude space (what a naive plot or a plate carrée projection gives you) or the true geodesic on the ellipsoid. These paths differ, and the difference is a real physical gap that an aircraft could fly through while your containment test says the opposite of the truth. How big is the gap?

```python
import numpy as np
from pyproj import Geod

geod = Geod(ellps="WGS84")

# An airspace edge drawn as a straight line in lat/lon vs the true geodesic.
# Both connect the same two vertices; how far apart are the paths at the midpoint?
def edge_deviation(lat, length_nm, azimuth=90.0):
    lon0 = -84.0
    lon1, lat1, _ = geod.fwd(lon0, lat, azimuth, length_nm * 1852.0)
    mid_chart = ((lon0 + lon1) / 2, (lat + lat1) / 2)   # straight-line midpoint
    mid_geo = geod.npts(lon0, lat, lon1, lat1, 3)[1]    # geodesic midpoint
    _, _, dev = geod.inv(mid_chart[0], mid_chart[1], mid_geo[0], mid_geo[1])
    return dev

print(f"{'edge length':>12s} {'deviation at KATL 33.6N':>25s} {'at Deadhorse 70.2N':>20s}")
for nm in [5, 15, 30, 60, 100]:
    print(f"{nm:9d} NM {edge_deviation(33.63, nm):22.1f} m {edge_deviation(70.19, nm):17.1f} m")
```

**Verified output:**
```
 edge length   deviation at KATL 33.6N   at Deadhorse 70.2N
        5 NM                    1.1 m               4.7 m
       15 NM                   10.0 m              41.9 m
       30 NM                   40.2 m             167.5 m
       60 NM                  160.8 m             669.9 m
      100 NM                  446.6 m            1861.1 m
```

The deviation grows roughly with the square of edge length, and it grows with latitude. At 15 NM, the edge length NASA's memo recommends as a ceiling, the two paths differ by 10 m near Atlanta, comfortably inside GPS-plus-buffer territory for our purposes. At 100 NM in Alaska the paths are almost 2 km apart, which is not a rounding error, it is a corridor an aircraft can occupy while two reasonable pieces of software disagree about whether it is inside controlled airspace.

The practical consequence for our pipeline: the FAA file densifies its polygons heavily (the KATL veil has over 800 vertices for a circle, spaced about a tenth of a degree apart), which keeps every edge short and makes the drawn-path question nearly moot for this dataset. But the moment you simplify a polygon to speed up rendering or containment tests, you are lengthening edges and buying back exactly this error. Simplification tolerance is therefore a safety parameter, not a performance knob, and it belongs in the assumption log with a number attached.

---

## 6. The Axis-Order Trap

One more trap, and it is the one most likely to burn you in the first week. The EPSG registry officially defines EPSG:4326 with coordinates ordered (latitude, longitude). GeoJSON, shapely, our airspace file, and essentially all flight data order them (longitude, latitude). pyproj honors the official order unless you tell it otherwise, and the failure mode is not an exception, it is coordinates on the far side of the planet.

```python
from pyproj import Transformer

klaf = (-86.9369, 40.4123)   # lon, lat of the Purdue University Airport ARP

t_default = Transformer.from_crs("EPSG:4326", "EPSG:32616")
t_xy      = Transformer.from_crs("EPSG:4326", "EPSG:32616", always_xy=True)

print("always_xy=False:", t_default.transform(*klaf))
print("always_xy=True: ", t_xy.transform(*klaf))
```

**Verified output:**
```
always_xy=False: (771670.8420965532, -10205864.477490613)
always_xy=True:  (505353.6628459792, 4473522.031208273)
```

The first line put Purdue's airport at a northing of negative ten million meters, which is deep in the southern hemisphere, because pyproj read our longitude as a latitude. The second line is correct: about 505 km east and 4474 km north in UTM 16N, which you can eyeball against a map. The rule is unconditional: **every `Transformer` in this project is constructed with `always_xy=True`, no exceptions**, and a code review that finds one without it has found a bug.

---

## 7. Applying This to the Real File

Everything above used a reconstructed circle so the tutorial is self-contained. Here is the pattern for the real data, which is the first block of the actual pipeline:

```python
import geopandas as gpd
from pyproj import CRS

gdf = gpd.read_file("Class_Airspace.geojson")     # loads as EPSG:4326 (CRS84)
katl = gdf[gdf["ICAO_ID"] == "KATL"]

# Build a local projection centered on the feature we are working on
c = katl.geometry.iloc[0].centroid
local = CRS.from_proj4(f"+proj=aeqd +lat_0={c.y} +lon_0={c.x} +datum=WGS84 +units=m")
katl_m = katl.to_crs(local)

print(katl_m.geometry.iloc[0].area / 1e6)          # now a real number, in km^2
```

Two checks to run every time this pattern appears. First, confirm the source CRS geopandas detected is what you expected, with `gdf.crs`; a file that arrives without CRS metadata gets assumed, and assumptions belong in the log. Second, after projecting, measure one known distance, the 30 NM veil radius, a published runway length, and compare against the authoritative number. In QGIS this is the measure tool over the reprojected layer; in code it is one `geod.inv` call. A projection bug caught by this check costs five minutes. The same bug caught after the STL is printed costs the print, the debugging session, and some confidence in everything else the pipeline produced.

---

## 8. Common Pitfalls

1. **Measuring in degrees and not noticing.** shapely will hand you `.area`, `.length`, and `.distance()` on unprojected geometry without complaint. If a number came out of shapely and the geometry was in EPSG:4326, the number has no unit.
2. **Reaching for Web Mercator because the coordinates were handy.** EPSG:3857 is for tile alignment. Its error is 44% at Atlanta, 72% at Purdue, and over 700% in northern Alaska.
3. **Forgetting `always_xy=True`.** The symptom is geometry mirrored, rotated, or in the wrong hemisphere. The fix is mechanical; make it a lint rule if you can.
4. **One UTM zone for a continent-scale batch job.** The airspace file spans zones 1 through 19. A batch pipeline either assigns each feature its own local projection or picks the UTM zone per feature centroid, never one zone for everything.
5. **Simplifying polygons without re-measuring edge error.** Section 5's deviation table is the price list. Every simplification tolerance is a claim that this much boundary error is acceptable, so write the claim down.
6. **Trusting `Shape__Area` and `Shape__Length` from the source file.** Both are in degree units. Recompute anything you intend to use.
7. **Projecting, editing, and forgetting which frame you are in.** Keep a convention: variables holding degree coordinates are suffixed `_deg` or live in clearly named GeoDataFrames, and geometry crosses the boundary through exactly one function. Mixed-frame bugs are the hardest ones in this project to see in a diff.

---

## 9. Exercises

1. The KLAF Class D surface area is listed in the file with its own `Shape__Area` in square degrees. Reconstruct its true area in km² two ways, geodesically and via a local azimuthal equidistant projection, and report the disagreement. (You should be able to get them within 0.01%.)
2. Find the airspace feature in the file whose centroid is furthest north. Compute the Web Mercator area error for that feature and compare it against the 44.65% we measured at Atlanta. Before running it, predict the number using 1/cos²(latitude).
3. Take the KATL veil and simplify it with shapely's `simplify()` at tolerances of 0.001, 0.01, and 0.05 degrees. For each, report the vertex count and the maximum distance between the original and simplified boundary in meters. At what tolerance would you stop trusting it as a geofence, and what buffer would make the 0.01 version acceptable?
4. Deadhorse, Alaska sits at 148.5°W. Which UTM zone contains it, and how far from the zone's central meridian is it? Compute the area error for a 30 NM circle there in the correct zone and in each neighboring zone.

---

## 10. Summary Table

| Question | Answer for this project |
|---|---|
| What CRS does the source data use? | EPSG:4326 / CRS84, coordinates in (lon, lat) degrees |
| Can I measure in it? | No. Degrees are not distances; areas come out in square degrees |
| What do I project into? | Local azimuthal equidistant centered on the airport being modeled |
| Acceptable general-purpose fallback? | Per-feature UTM zone, chosen by centroid, noted in the assumption log |
| What is never acceptable for measurement? | Web Mercator, EPSG:3857 |
| Who measures true distance and area? | `pyproj.Geod` on the WGS84 ellipsoid |
| Who does the geometry? | shapely, only after projection to meters |
| The flag that must always be set? | `always_xy=True` on every Transformer |
| The check after every projection? | Re-measure one known distance against its published value |

---

## 11. Further Reading

Start with the pyproj documentation's "Gotchas" page, which covers the axis-order issue and several relatives in about ten minutes of reading, and the shapely manual's opening section, which states plainly that the library is planar and unit-agnostic.

Neeley, P. and Narkawicz, A. (2017), *Map Projection Induced Variations in Locations of Polygon Geofence Edges*, NASA/TM-2017-219675, free on NTRS, is the paper behind Section 5 and directly motivates the vertex-distance and edge-length limits for geofencing. It is short and readable.

For a broader foundation, Snyder's *Map Projections: A Working Manual* (USGS Professional Paper 1395, free from USGS) is the classic reference; Chapter 1 alone covers everything this tutorial used. The epsg.io website is the fastest way to look up any CRS you encounter.

For the ENU connection to the GNC side of the program, Groves, *Principles of GNSS, Inertial, and Multisensor Integrated Navigation Systems*, Chapter 2, treats Earth-centered and local tangent-plane frames with the same math we used for the azimuthal equidistant projection, and makes the correspondence exact.

Next tutorial in this series: **Polygons to Solids: Extrusion, Triangulation, and Mesh Validity**, where the projected footprints from this lesson get pulled upward into watertight prisms.
