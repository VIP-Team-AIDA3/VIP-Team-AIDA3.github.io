# Introduction to GIS: Maps, Layers, and the Data Behind Them

*A first tutorial on geographic information systems, using the airspace and airport data our project is built on.*

You don't need any prior exposure to GIS for this. If you have used Google Maps, you have already used one.

## What this is about

Everything in the geospatial airspace project, the extruded solids, the runway footprints, the live traffic overlay, is built from a small set of ideas that the GIS field settled decades ago: data organized as layers, geometry paired with a table of attributes, a distinction between vector and raster data, and operations that combine layers to answer questions. This tutorial introduces those ideas using the two files the project actually runs on, so by the end you will have queried real FAA data and know what the words in the rest of the series mean.

This material draws on the Purdue Lyles School of Civil Engineering "Introduction to GIS" lecture and the MIT OpenCourseWare GIS Level 1 and Level 2 workshops (RES.STR-001), both listed in the reading section, and adapts their content to our aviation setting.

---

## Table of Contents

1. [What a GIS Is](#1-what-a-gis-is)
2. [Layers: How a Map Is Assembled](#2-layers-how-a-map-is-assembled)
3. [Vector and Raster: The Two Kinds of Spatial Data](#3-vector-and-raster-the-two-kinds-of-spatial-data)
4. [The Attribute Table: Where Half the Information Lives](#4-the-attribute-table-where-half-the-information-lives)
5. [Selecting and Joining: The Two Queries Behind Everything](#5-selecting-and-joining-the-two-queries-behind-everything)
6. [File Formats You Will Meet](#6-file-formats-you-will-meet)
7. [Coordinate Systems, Briefly](#7-coordinate-systems-briefly)
8. [The Standard Processing Tools](#8-the-standard-processing-tools)
9. [Software: QGIS, ArcGIS Pro, and Python](#9-software-qgis-arcgis-pro-and-python)
10. [Metadata and Data Sources](#10-metadata-and-data-sources)
11. [Making Readable Maps](#11-making-readable-maps)
12. [Common Pitfalls](#12-common-pitfalls)
13. [Exercises](#13-exercises)
14. [Summary Table](#14-summary-table)
15. [Further Reading](#15-further-reading)

---

## 1. What a GIS Is

The standard definition, which both the Purdue and MIT introductions use in nearly the same words, is a system for capturing, storing, checking, integrating, manipulating, analyzing, and displaying spatial data. That is a long list, but it reduces to one sentence: a GIS is a database whose records have a location, plus tools that use the locations.

The database part matters more than beginners expect. A working definition you will hear in the field is that GIS is three things at once: a set of software tools, a database for geographic information, and a decision-support system that integrates spatially referenced data into a problem-solving workflow. Our project is all three. The FAA files are the database, shapely and QGIS are the tools, and the geofence query for Earhart is the decision the whole pipeline supports.

One more framing worth keeping. The MIT workshop points out that GIS software does not come with data; it is an empty analysis engine until you feed it. Most of the actual work in any GIS project, ours included, is finding, understanding, cleaning, and trusting the data, and only then analyzing it.

---

## 2. Layers: How a Map Is Assembled

Open Google Maps and look at what is on screen: a street network, water bodies, parks, transit stops, labels, and a satellite image underneath if you toggled it. Each of those arrived as a separate dataset, was drawn in a fixed order, and could be turned on or off independently. That is the layer model, and every GIS uses it.

Our project's scene decomposes the same way. From the bottom up: a basemap or satellite imagery layer for context, a terrain elevation layer, the runway footprint layer built from the FAA airport table, the airspace polygon layer from `Class_Airspace.geojson`, a temporary flight restriction layer that updates on a timer, and an aircraft layer that updates every few seconds. Each layer has its own source, its own update cadence, and its own symbology, and the analysis operations in Section 8 work by combining them. When you sketch a new feature for the viewer, the first design question is which layer it belongs to and where in the stack it draws.

---

## 3. Vector and Raster: The Two Kinds of Spatial Data

Spatial data comes in two representations, and the choice between them shapes what you can do.

**Vector** data stores geometry as coordinates: points, lines, and polygons. An airport reference point is a point. A runway centerline is a line. A Class D surface area is a polygon. Vector data suits things with defined boundaries, usually the built and regulated world. Both of our project files are vector data.

**Raster** data stores a grid of cells, each holding a value, like a photograph where every pixel means something. Elevation models, satellite imagery, weather radar mosaics, and scanned charts are rasters. Raster suits continuous phenomena, things that vary smoothly across space and have no natural boundary. The digital elevation model we will use to drape terrain-following airspace floors in Tutorial 3 is a raster, and the georeferenced imagery the vision subteam trains on is raster data too.

The two convert into each other when needed. Contour lines are vectors extracted from an elevation raster; a rasterized airspace mask is a raster burned from our polygons. A useful habit when you meet a new dataset is to ask which kind it is before anything else, because the answer determines which tools apply.

---

## 4. The Attribute Table: Where Half the Information Lives

A vector layer is two things stapled together: the geometry you see on the map, and a table with one row per feature, called the attribute table. Every polygon in the airspace file has a row carrying its identifier, class, floor, ceiling, and about thirty other columns. The map and the table are two views of the same features, and clicking a polygon in QGIS highlights its row, and the reverse.

We can open the attribute table without touching the geometry at all, since the FAA also publishes it as a plain CSV:

```python
import pandas as pd

# The attribute table behind the airspace layer, without the geometry
air = pd.read_csv("Class_Airspace.csv", encoding="utf-8-sig")
print(f"features: {len(air)}")
print(air["CLASS"].value_counts().to_string())
print()
print(air.loc[air["ICAO_ID"] == "KLAF",
      ["IDENT", "NAME", "CLASS", "LOWER_VAL", "LOWER_CODE", "UPPER_VAL", "UPPER_CODE"]].to_string(index=False))
```

**Verified output:**
```
features: 6054
CLASS
E        4340
D         649
B         423
C         411
A          11
Other       8
G           2

IDENT               NAME CLASS  LOWER_VAL LOWER_CODE  UPPER_VAL UPPER_CODE
  LAF  LAFAYETTE CLASS D     D          0        SFC       3100        MSL
  LAF LAFAYETTE CLASS E2     E          0        SFC      -9998        NaN
```

Two rows for our home airport, and both matter to the project. The Class D runs from the surface to 3,100 ft MSL, exactly the prism we will extrude in Tutorial 2. The Class E2 has a ceiling of -9998, the null sentinel discussed in the project brief, and here it is in the wild on the first airport we look at. Reading attribute tables carefully, including their sentinel values and their units, is where data problems get caught, and it costs nothing compared to catching them after the geometry is built.

---

## 5. Selecting and Joining: The Two Queries Behind Everything

GIS analysis rests on two query types. **Selection by attribute** filters features by their table values, the spatial equivalent of a SQL WHERE clause. **Joining** attaches one table to another through a shared key, so information collected separately can be used together. Both work the same whether you run them in QGIS, ArcGIS, or pandas.

A selection, phrased as an operational question: which Indiana runways could our fixed-wing operations realistically use?

```python
import pandas as pd

rwy = pd.read_csv("us_airports_-_us_airports.csv", encoding="utf-8-sig")

# Select by attribute: Indiana runways, paved, at least 5000 ft
sel = rwy[(rwy["STATE_CODE"] == "IN")
          & (rwy["SURFACE_TYPE_CODE"].isin(["ASPH", "CONC", "ASPH-CONC"]))
          & (rwy["RWY_LEN"] >= 5000)]

print(f"Indiana runway records:            {len(rwy[rwy['STATE_CODE']=='IN'])}")
print(f"paved and >= 5000 ft:              {len(sel)}")
print(f"airports meeting the requirement:  {sel['ARPT_ID'].nunique()}")
print()
print(sel[["ARPT_ID", "ARPT_NAME", "RWY1_ID", "RWY2_ID", "RWY_LEN", "SURFACE_TYPE_CODE"]]
      .sort_values("RWY_LEN", ascending=False).head(8).to_string(index=False))
```

**Verified output:**
```
Indiana runway records:            193
paved and >= 5000 ft:              54
airports meeting the requirement:  45

ARPT_ID         ARPT_NAME RWY1_ID RWY2_ID  RWY_LEN SURFACE_TYPE_CODE
    GUS       GRISSOM ARB       5      23    12501              ASPH
    FWA   FORT WAYNE INTL       5      23    11981         ASPH-CONC
    IND INDIANAPOLIS INTL       5      23    11200              CONC
    IND INDIANAPOLIS INTL       5      23    10000              CONC
    HUF  TERRE HAUTE RGNL       5      23     9021              ASPH
    GYY GARY/CHICAGO INTL      12      30     8859              CONC
    SBN   SOUTH BEND INTL       9      27     8412              ASPH
    EVV   EVANSVILLE RGNL       4      22     8021              ASPH
```

Notice that IND appears twice with identical runway designators on both rows, which is unlikely for parallel runways that should be 5L/23R and 5R/23L. That is a data quality observation, and logging observations like it is part of the job.

Now a join, connecting our two files through their shared airport identifier: which Indiana airports sit inside their own Class D airspace?

```python
import pandas as pd

air = pd.read_csv("Class_Airspace.csv", encoding="utf-8-sig")
rwy = pd.read_csv("us_airports_-_us_airports.csv", encoding="utf-8-sig")

# Tabular join on a shared key: which Indiana airports have their own Class D?
ind_airports = rwy.loc[rwy["STATE_CODE"] == "IN", ["ARPT_ID", "ARPT_NAME"]].drop_duplicates()
class_d = air.loc[air["CLASS"] == "D", ["IDENT", "NAME", "UPPER_VAL"]]

joined = ind_airports.merge(class_d, left_on="ARPT_ID", right_on="IDENT", how="inner")
print(f"Indiana airports in the runway table: {ind_airports['ARPT_ID'].nunique()}")
print(f"of which have a Class D of their own: {joined['ARPT_ID'].nunique()}")
print()
print(joined[["ARPT_ID", "ARPT_NAME", "NAME", "UPPER_VAL"]].to_string(index=False))
```

**Verified output:**
```
Indiana airports in the runway table: 148
of which have a Class D of their own: 9

ARPT_ID            ARPT_NAME                NAME  UPPER_VAL
    AID        ANDERSON RGNL    ANDERSON CLASS D       3400
    BMG        MONROE COUNTY BLOOMINGTON CLASS D       3300
    BAK        COLUMBUS MUNI    COLUMBUS CLASS D       3200
    EKM         ELKHART MUNI     ELKHART CLASS D       3300
    GYY    GARY/CHICAGO INTL        GARY CLASS D       3100
    LAF    PURDUE UNIVERSITY   LAFAYETTE CLASS D       3100
    MIE DELAWARE COUNTY RGNL      MUNCIE CLASS D       3400
    GUS          GRISSOM ARB GRISSOM ARB CLASS D       3300
    HUF     TERRE HAUTE RGNL TERRE HAUTE CLASS D       3100
```

Nine towered fields in the state, LAF among them, joined out of 148 airports with nothing but a shared identifier column. There is a third query type this tutorial only names: **selection by location**, where the filter is spatial rather than tabular, for example every obstacle inside a Part 77 surface. That one needs geometry and a projection, which is why it waits for Tutorials 1 and 4.

---

## 6. File Formats You Will Meet

**GeoJSON** is what our airspace data ships as: a single text file, human-readable, with geometry and attributes together. It is the working format for this project and the native format of the web viewers we will build in.

**The shapefile** is the format you will meet everywhere else in the GIS world. Despite the singular name it is a bundle of files with the same base name and different extensions (.shp for geometry, .dbf for the attribute table, .prj for the coordinate system, .shx as an index, sometimes more), and all of them must travel together. A shapefile missing its .prj is a dataset that has forgotten where on Earth it is, and someone will have to guess.

**KML** is Google Earth's format, and our airspace data ships in it as well. Its practical role in this project is visual checking, since dropping a KML onto Google Earth over imagery takes seconds.

**CSV** carries attributes with no geometry, or with geometry hiding in coordinate columns. Both files used in this tutorial are CSVs, and turning a coordinate-column CSV into a point layer is a one-step operation in any GIS.

**GeoTIFF** is the raster workhorse, an image format with georeferencing baked in; the elevation models in Tutorial 3 arrive this way. Beyond these, GIS software converts most things into most other things: CAD drawings, GPX tracks from a GPS unit, LAS point clouds from lidar, and NetCDF scientific data all import, which is often the whole reason GIS sits in the middle of a workflow.

---

## 7. Coordinate Systems, Briefly

Every spatial dataset answers the question "where" in some agreed reference frame, and there are two families of them. A geographic coordinate system locates points with angles, latitude and longitude in degrees, on a model of the curved Earth. A projected coordinate system flattens a region of that curved surface onto a plane so that coordinates become linear units, meters or feet, that support honest measurement. Data usually arrives geographic and gets projected before analysis; any tool involving distance, area, or direction expects projected input.

That is the entire summary, because Tutorial 1 in this series is devoted to the subject, including what goes wrong when it is skipped. For now, one habit: the first thing to check on any new dataset is its coordinate system, in QGIS under layer properties, in code via `gdf.crs`. If the answer is missing or surprising, stop and resolve it before doing anything else.

---

## 8. The Standard Processing Tools

The MIT Level 2 workshop organizes spatial analysis into a small vocabulary of tools that recur under different names in every package. The ones this project will actually use:

**Buffer** grows a geometry outward by a distance, turning a point into a disk or a polygon into a fattened polygon. Our safety margins are buffers: an airspace boundary buffered inward by a GPS-uncertainty distance is a conservative geofence.

**Clip** cuts one layer down to the extent of another, like a cookie cutter. Cropping the national airspace file to a study area around KLAF is a clip, and its raster twin (extract by mask) crops the elevation model the same way.

**Intersect and union** are the polygon overlay operations, producing the shared region or the combined region of two layers. Checking whether a proposed flight corridor overlaps a temporary flight restriction is an intersection.

**Dissolve** merges features that share an attribute value into one geometry, which is how the many separate shelf polygons of a Class B get reassembled into a single airport's structure.

**Zonal statistics** summarizes a raster inside each polygon of a vector layer, one row of statistics per zone. Asking for the minimum, mean, and maximum terrain elevation under each SFC-floored airspace polygon, which Tutorial 3 needs, is zonal statistics verbatim.

**Interpolation, slope, and contour** operate on surfaces: building a continuous surface from scattered points, computing steepness per cell, and extracting lines of constant value. These appear when terrain enters the pipeline.

Two pieces of advice from the MIT material apply directly. The accuracy of a tool's output is capped by the accuracy of its input, so garbage geometry in means garbage analysis out. And every tool run should be recorded, ideally as code, so results can be replicated, which is the reason our project rule says deliverables come from scripts in the repository rather than from clicks.

---

## 9. Software: QGIS, ArcGIS Pro, and Python

Three tiers of software do GIS work, and this project touches all of them.

**QGIS** is free, open source, runs on any operating system, and covers everything this project needs from a desktop application. It is our tool for looking at data: checking a CRS, clicking features to read attributes, measuring a known distance after a projection, and validating geometry. Purdue teaches with ArcGIS Pro in several courses, and if you already know it, the concepts transfer completely; the main practical differences are that ArcGIS Pro is commercial, Windows-only, and stronger in a few specialized areas like network analysis, while QGIS is free and everywhere. One vocabulary difference worth knowing: ESRI software names coordinate systems in words, while QGIS uses EPSG codes, so "WGS 1984" and "EPSG:4326" are the same thing wearing different badges.

**Python** with geopandas, shapely, and pyproj is where our production pipeline lives, for the reason given in Section 8: code is repeatable and reviewable, clicks are not. The same buffer, clip, and join operations exist as function calls, and everything a deliverable contains must be generated this way.

**Web viewers**, CesiumJS or deck.gl for us, are the delivery tier, where processed layers become something a person can click. The layer model of Section 2 carries straight through: the viewer is a layer stack with a camera.

The division of labor, stated once and worth remembering: QGIS to inspect, Python to produce, the browser to present.

---

## 10. Metadata and Data Sources

Metadata is the documentation that rides with a dataset: who made it, when, from what sources, in which coordinate system, with what accuracy, and what each attribute column means. The FAA publishes exactly this for our files, and the difference between guessing what `UPPER_CODE` means and knowing came from reading it. When a dataset arrives without metadata, treat every property you assume about it as a claim you now owe evidence for.

Knowing where authoritative data lives is a skill in itself. For this project the sources are the FAA's aeronautical data portal for airspace, airports, runways, and obstacles, published on a 28-day cycle, which matters because it means our snapshot has a date and will drift; the USGS 3DEP program for national elevation models; and state GIS portals, IndianaMap for us, for imagery and local layers. OpenStreetMap covers nearly everything else and is the usual starting point where official data is thin. The general search advice from the MIT workshop holds: look for the agency that has a legal mandate to maintain the data, and prefer it over anything repackaged.

---

## 11. Making Readable Maps

Cartography is a full discipline and this section will not attempt it, but three rules prevent most bad maps, and all three come up the first week someone styles the viewer.

Match the color scheme to the data type. Categorical data, like airspace class, gets distinct hues with no implied order. Quantitative data, like ceiling altitude, gets a sequential ramp running light to dark. Data diverging around a meaningful middle gets a two-direction ramp. ColorBrewer (colorbrewer2.org) exists to hand you defensible palettes, including colorblind-safe ones, and there is no reason to invent your own.

Symbolize for the scale you are at. A runway is a polygon on an airport diagram and a line, or nothing, on a state map. Decluttering is not dishonesty; it is the difference between a map and a data dump.

Answer three questions before styling anything, which the MIT workshop poses as the whole design method: who is the map for, where will it be seen, and what is it supposed to show. A conference poster, a debugging screenshot, and the live operations viewer want different maps of the same data.

---

## 12. Common Pitfalls

1. **Treating the map as the data.** The attribute table is half the dataset, and usually the half where problems hide. Open it first.
2. **Ignoring sentinel values.** The -9998 ceilings in the airspace file are the local example. Every agency has its own null conventions, and none of them will throw an exception for you.
3. **Separating a shapefile from its sidecar files.** Copying only the .shp breaks the dataset, and losing the .prj silently costs you the coordinate system.
4. **Assuming a join key is clean.** Airport identifiers come in FAA and ICAO flavors (LAF versus KLAF), and joining across the two without normalizing quietly drops rows. Count records before and after every join.
5. **Doing analysis in a geobrowser.** Google Earth and web maps display data; they do not measure it defensibly. Tools that compute belong in QGIS or Python.
6. **Skipping metadata.** Ten minutes with the FAA's data dictionary saves the afternoon you would spend reverse-engineering what a column means from its values.
7. **Clicking out results.** An analysis produced by hand in a GUI cannot be rerun, reviewed, or trusted six weeks later. If it appears in a deliverable, it comes from code.

---

## 13. Exercises

1. Using the airspace CSV, count how many features have `LOWER_CODE` equal to SFC, and separately how many have `UPPER_VAL` equal to -9998. What fraction of the file will the terrain-drape and null-ceiling decisions from the project brief actually touch?
2. Extend the Section 5 selection to find every U.S. airport with a turf runway of at least 3,000 ft. How many are in Indiana, and which is closest to Purdue by coordinates?
3. Rerun the Section 5 join using `ICAO_ID` from the runway table against the airspace file instead of `ARPT_ID` against `IDENT`. Compare the row counts and explain the difference. (This is pitfall 4 made concrete.)
4. Load `Class_Airspace.geojson` into QGIS. Identify the layer's CRS, open the attribute table, select the Indianapolis Class C features, and zoom to them. Using only the identify tool, sketch the shelf structure on paper: how many rings, and what are their floors and ceilings?
5. Find the FAA's published data dictionary for the airport CSV and write down, with a citation, what `RWY_LEN_SOURCE` means and what units `ELEV` uses. This is a metadata retrieval exercise, and the answer must come from the source document rather than a guess.

---

## 14. Summary Table

| Concept | Meaning | Where it appears in our project |
|---|---|---|
| Layer | Independently drawn, ordered dataset in a map | Terrain, runways, airspace, TFRs, traffic |
| Vector | Geometry as coordinates: points, lines, polygons | Both FAA files |
| Raster | Grid of valued cells | Elevation model, training imagery |
| Attribute table | One row of properties per feature | Floors, ceilings, classes, runway lengths |
| Select by attribute | Filter features by table values | Paved runways over 5,000 ft |
| Join | Attach tables via a shared key | Airports matched to their Class D |
| Buffer / clip / dissolve | Core geometry operations | Safety margins, study areas, Class B reassembly |
| Zonal statistics | Raster summarized per polygon | Terrain under SFC floors |
| Metadata | Documentation riding with data | FAA data dictionary, 28-day cycle date |
| QGIS vs Python | Inspect vs produce | Verification vs the pipeline |

---

## 15. Further Reading

The two sources this tutorial adapts are worth reading in full. The Purdue "Introduction to GIS" lecture (Lyles School of Civil Engineering) surveys the field, its software, and the GIS programs available on campus, including the Graduate Certificate in Geospatial Information Science and the EAPS Geodata Science professional master's. The MIT OpenCourseWare GIS Tutorial (RES.STR-001, IAP 2022) is free, includes hands-on QGIS exercises with data, and its two workshops map onto this tutorial directly: Level 1 covers layers, data types, formats, and cartography, and Level 2 covers projections and the processing tools. Doing the MIT Level 1 QGIS exercise is the single best follow-up to this lesson, and takes about two hours.

For a book, Bolstad's *GIS Fundamentals* is the standard comprehensive text and covers everything here at real depth. The QGIS Training Manual (docs.qgis.org) is free and exercise-driven. For the cartography section, ColorBrewer at colorbrewer2.org is a tool rather than a reading, and Krygier and Wood's *Making Maps* is the approachable design reference the MIT workshop draws its examples from.

The FAA side has its own documents: the aeronautical data portal's data dictionaries define every column in our two files, and exercise 5 sends you there deliberately.

Next tutorial in this series: **From Degrees to Meters: Coordinate Reference Systems and Projection Error**, which takes the one-paragraph summary in Section 7 and turns it into working code.
