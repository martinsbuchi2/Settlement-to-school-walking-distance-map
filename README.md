# Settlement-to-School Walking Distance Map

## Project Overview

This project identifies settlements across Ghana that fall beyond practical walking distance of any school, defined as a 5-kilometre straight-line threshold. Every settlement is evaluated against the full education facility network, tagged with a school proximity class, and a filtered subset of underserved settlements is exported as a standalone layer. The outputs are designed to directly support education infrastructure planning, district assembly investment decisions, and NGO intervention targeting in communities where access to formal education is constrained by distance.

---

## Coordinate Reference System

All layers are reprojected to **EPSG:32630 (WGS 84 / UTM Zone 30N)** before processing. The source layers (education_facilities, settlements, roads) were originally in EPSG:25000. Reprojection to a metric UTM zone is mandatory as the 5,000-metre distance threshold requires accurate planar distance calculations.

---

## Input Layers

| Layer | File (reprojected) | Geometry | CRS | Features |
|---|---|---|---|---|
| Education facilities | education_facilities_32630.gpkg | Point | EPSG:32630 | 2,912 |
| Settlements | settlements_32630.gpkg | Point | EPSG:32630 | 25,342 |
| Roads (context) | roads_32630.gpkg | Line | EPSG:32630 | 374,753 |

---

## Processing Workflow

### Step 1 — Reproject all layers to EPSG:32630
Each of the three source layers was processed with **Reproject Layer** (native:reprojectlayer), targeting EPSG:32630. The reprojected files carry the `_32630` suffix to distinguish them from their source equivalents.

### Step 2 — Generate 5 km dissolved school buffer
A dissolved buffer of **5,000 metres** was applied to `education_facilities_32630.gpkg` using **Buffer** (native:buffer) with 16 segments per quadrant and Dissolve result enabled. This produces a single unified polygon representing the full on-threshold school service zone. Output saved as `school_5km_buffer.gpkg`.

### Step 3 — Join each settlement to its nearest school
**Join Attributes by Nearest** (native:joinbynearest) was applied with `settlements_32630.gpkg` as the input and `education_facilities_32630.gpkg` as the join layer. Parameters: 1 nearest neighbour, no maximum distance cap. The output carries school attribute fields (prefixed `school_`) and a `distance` field in metres. Output saved as `settlements_school_dist.gpkg`.

### Step 4 — Tag settlements with school access class
A **Field Calculator** pass (native:fieldcalculator) added the field `school_access` (text, 20 characters):

```
if("distance" <= 5000, 'Within 5km', 'Beyond 5km')
```

Output saved as `settlements_school_tagged.gpkg`. Feature count is identical to the input (25,342), preserving full dataset integrity.

### Step 5 — Extract beyond-5km settlements
**Extract by Attribute** (native:extractbyattribute) filtered `settlements_school_tagged.gpkg` on `school_access = 'Beyond 5km'`, producing the standalone priority layer `settlements_beyond_5km.gpkg`.

---

## Output Layers

| Layer | File | Features | Key Fields |
|---|---|---|---|
| School 5 km service zone | school_5km_buffer.gpkg | 1 | (dissolved polygon) |
| Settlements with school distance | settlements_school_dist.gpkg | 25,342 | village_name, school_name, distance |
| Settlements tagged by access class | settlements_school_tagged.gpkg | 25,342 | village_name, school_name, distance, school_access |
| Settlements beyond 5 km (priority layer) | settlements_beyond_5km.gpkg | 16,726 | village_name, school_name, distance, school_access |

---

## Key Findings

| Access class | Settlements | Share of total |
|---|---|---|
| Within 5 km of a school | 8,616 | 34.0% |
| Beyond 5 km from any school | 16,726 | 66.0% |
| Total settlements | 25,342 | 100% |

Two-thirds of all settlements in Ghana (16,726 of 25,342) fall beyond 5 km of any school. This represents the primary analytical finding of the project and the direct input for prioritising new school construction, satellite learning centres, or transport improvement interventions.

---

## Symbology

`settlements_school_tagged.gpkg` uses a **categorised renderer** on `school_access`: blue circles (size 1.4) for Within 5km settlements and red circles (size 1.4) for Beyond 5km settlements. `education_facilities_32630.gpkg` is displayed as green diamonds (size 3.0) with a dark green outline. `roads_32630.gpkg` is shown as thin grey lines (width 0.15) for transport context. `school_5km_buffer.gpkg` is rendered as a semi-transparent yellow fill with an amber outline, communicating the service threshold zone visually.

---

## Project File

`Settlement-to-school walking distance map.qgz` — saved with relative paths (Qgis.FilePathType.Relative). All layers load without broken links on any machine that preserves the folder structure.
