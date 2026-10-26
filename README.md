# Roadway Maintenance — Geospatial Data Integration & Analysis

## 1. Project Overview

This project integrates Washington, DC construction permit records, Cityworks roadway service requests, and roadway subblock centerlines to support maintenance planning and operational decision-making.

The objective was to identify roadway subblocks containing both qualifying construction permit renewals and roadway maintenance requests, calculate associated counts and latest activity dates, and provide an interactive GIS project for further exploration.

The analysis was completed using **QGIS**, with local GeoPackage datasets and geoprocessing history included in the submission.

## 2. Key Results

| Metric                                |  Result |
| ------------------------------------- | ------: |
| Eligible roadway subblocks            |  23,104 |
| Qualifying construction permits       |   8,638 |
| Qualifying Cityworks service requests |     698 |
| Subblocks with service requests       |     588 |
| **Subblocks containing both**         | **106** |

The 106 overlapping subblocks represent locations where construction permit activity and roadway maintenance requests coexist, providing potential areas for further operational review.

These results reflect the source data and documented roadway-assignment methodology used in this submission.

## 3. Data Sources and Filtering

Three DC GIS REST services were used:

- **Construction Permits:** [DDOT TOPS — Layer 23](https://maps2.dcgis.dc.gov/dcgis/rest/services/DCGIS_APPS/DDOT_TOPS/MapServer/23)
- **Roadway Subblocks:** [Transportation — Layer 162](https://maps2.dcgis.dc.gov/dcgis/rest/services/DCGIS_DATA/Transportation_WebMercator/MapServer/162)
- **Cityworks Service Requests:** [Cityworks — Layer 1](https://maps2.dcgis.dc.gov/dcgis/rest/services/DDOT/Cityworks/MapServer/1)

Records were filtered as follows:

| Dataset              | Criteria                                                                                                               |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Construction Permits | Issued renewals, excavation or paving, expiration on/after October 1, 2025                                             |
| Roadway Subblocks    | Streets, service roads, and ramps                                                                                      |
| Cityworks            | Specified roadway repair/pothole/paving descriptions, eligible nonclosed statuses, initiation on/after October 1, 2025 |

The source classification *Service Road* was interpreted as the assignment's *Service Lane* category.

The original REST connections remain in the QGIS project, alongside local analytical datasets.

## 4. Processing Methodology

### Data Preparation

The three filtered datasets were exported to local GeoPackage files. Their original EPSG:3857 coordinates were transformed to EPSG:26985 (NAD83 / Maryland) to support distance-based geospatial analysis.

An available coordinate transformation with a stated accuracy of 4 meters was used because the preferred 2-meter transformation required an unavailable grid file (`us_noaa_MD.tif`).

### Permit Location Assessment and Snapping

Initial nearest-roadway analysis identified substantial variation in permit location accuracy:

| Original distance to nearest roadway | Permit count |
| ------------------------------------ | -----------: |
| 0–5 meters                           |        7,629 |
| 5–20 meters                          |          149 |
| 20–50 meters                         |          817 |
| Over 50 meters                       |           43 |
| **Total**                            |    **8,638** |

The initial 7,629 permits and the following 149 permits were snapped in separate tolerance-based operations. Post-snapping distance checks confirmed effectively zero geometric offset for these groups. Roadway identifier consistency was also verified for the 149-permit group.

For the 817 permits located 20–50 meters away, the two nearest roadway candidates were compared. Of these, 513 had a distance difference of 2 meters or less, indicating potential assignment ambiguity.

Further inspection revealed cases where different subblocks belonged to the same street, as well as cases where the nearest street did not match the permit's work-location address.

A complete snapped dataset containing all 8,638 permits was subsequently generated using the nearest points on the matched roadway geometries. Original distance measurements and review classifications were retained.

**The completed dataset is geometrically snapped, but some roadway assignments remain provisional.** In particular, the 860 permits originally over 20 meters from their nearest road require additional location verification.

### Roadway Enrichment

Construction permits and Cityworks service requests were associated with roadway subblocks using nearest-roadway relationships and roadway identifiers.

Records were aggregated by roadway subblock to calculate:

- `permit_count` — Number of qualifying permits.
- `permit_max` — Latest permit expiration date.
- `request_count` — Number of qualifying service requests.
- `request_max` — Latest request initiation date.
- `has_both` — Binary indicator identifying blocks with both permits and requests.
- `overlap_status` — Human-readable overlap classification, where included.

Subblocks without matching records were retained, with zero counts and null dates as appropriate.

## 5. Validation

The following checks were performed:

| Validation                                       | Result                             |
| ------------------------------------------------ | ---------------------------------- |
| Final roadway feature count                      | 23,104                             |
| Total aggregated permit count                    | 8,638                              |
| Total aggregated request count                   | 698                                |
| Both-present subblock count                      | 106                                |
| Snapped permit feature count                     | 8,638                              |
| Snapping geometry checks for first 7,778 permits | Effectively zero residual distance |

These checks establish record-count completeness and internal aggregation consistency. They do not independently certify every nearest-roadway assignment.

## 6. Deliverables

The QGIS project includes:

- Enriched roadway subblock centerlines with calculated attributes.
- A locally stored, snapped construction permit dataset.
- Filtered Cityworks service request locations.
- The three original ArcGIS REST service connections.
- Intermediate geoprocessing outputs and exported processing history.
- Original filtered and projected local datasets.

The map highlights roadway subblocks where `has_both = 1`, allowing reviewers to explore overlapping roadway activity.

The QGIS project is stored in `.qgz` format, and local analytical data is stored in GeoPackage (`.gpkg`) format, the QGIS-compatible alternative to an ArcGIS file geodatabase.

## 7. Challenges, Limitations, and Further Improvements

**Roadway assignment ambiguity:** Geographic proximity does not always identify the intended roadway. For example, one inspected permit on Newton Street NW was geographically closer to Monroe Street NW. This demonstrated the need to incorporate work-location addresses and roadway attributes alongside spatial distance.

**Subblock ambiguity:** Multiple centerline subblocks can be equally close to a permit, including adjacent segments of the same route. This may affect subblock-level allocation even when the roadway name is correct.

**Transformation accuracy:** The installed coordinate transformation was less accurate than the preferred grid-based alternative. Additional accuracy assessment would be appropriate before operational deployment.

**Additional work with more time:**

1. Develop an automated matching procedure combining normalized street addresses, roadway names, candidate distances, and available segment identifiers.
2. Resolve ambiguous subblock assignments with network topology, address geocoding, and manual verification of high-risk cases.
3. Revalidate every snapped permit against its intended roadway subblock and quantify any changes to the final overlap counts.
4. Automate the full workflow using PyQGIS or standalone Python scripts, including repeatable quality checks and exception reports.
5. Review the City's source status definitions and data refresh practices to ensure the service-request population accurately reflects operational requirements.

The present outputs are suitable for exploratory analysis and demonstration of the requested GIS workflow. Final operational use should be preceded by verification of the flagged roadway assignments.

## 8. Reproducing and Exploring the Project

1. Extract the supplied ZIP archive while preserving the directory structure.
2. Open `DC_Roadway_Analysis.qgz` in QGIS.
3. Confirm the local GeoPackage layers are accessible.
4. Enable `roadway_enriched`, `permits_snapped_final`, and dc\_`cityworks_projected`.
5. Filter `roadway_enriched` using `has_both = 1` to examine overlapping blocks.
6. Review the intermediate GeoPackage outputs and processing history for details of the geoprocessing workflow.

The live REST layers require an internet connection.

---

**Conclusion**

The project demonstrates the integration of multiple public GIS services into a roadway-level analytical dataset, including spatial matching, geometry correction, aggregation, quality assessment, and interactive visualization.

The analysis identified **106 roadway subblocks with both qualifying construction permits and Cityworks service requests**, while transparently documenting the location-quality limitations that would require additional attention before production use.

