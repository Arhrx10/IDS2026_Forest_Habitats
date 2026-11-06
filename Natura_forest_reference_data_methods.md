# Preparation of Natura Forest Habitat Reference Data

Monitoring and inventory data are prepared for later remote-sensing modelling. Records from 20 m radius monitoring plots are cleaned, spatially separated training and evaluation groups are created, and optional additional points can be sampled inside habitat inventory polygons.

## Monitoring data preparation

Records from KESE forest habitat monitoring are selected when a 20 m plot radius is explicitly recorded. As each indicator is stored in a separate CSV row, the indicators belonging to one observation are combined into a single record. Habitat type is taken from the *Elupaigatüüp (LD)* field. Habitat codes and assessment categories are standardised, while the original values are retained for checking. The reversed easting and northing fields in the source file are corrected, and observations with invalid coordinates are separated. Observations are flagged according to whether they fall within 2017–2026, but other years are not automatically removed. A circular polygon with a 20 m radius is also created around each valid monitoring centre.

## Inventory data preparation

Natura habitat inventory polygons are loaded if the inventory file is available. Their geometries are transformed to EPSG:3301, invalid geometries are repaired, and unusable polygons are excluded. Relevant field names and assessment codes are standardised while retaining the original values. An inventory assessment is understood to describe the entire polygon, rather than a measured condition at one specific point.

## Spatial splitting and leakage control

Monitoring plots are grouped into 10 × 10 km spatial blocks. Five folds are created using `StratifiedGroupKFold`, which keeps plots from the same block together and attempts to balance the selected assessment classes. By default, the balance is based on structural preservation. Fold 0 is reserved for final testing, while folds 1–4 are used for training and spatial cross-validation.

Where monitoring plots overlap inventory polygons, their fold assignments are coordinated. Polygons associated with plots from different blocks or folds, together with the affected monitoring plots, are flagged for exclusion. Training observations less than 1 km from final test observations are also flagged for exclusion, but the records are retained for quality control. A similar distance buffer can be applied during cross-validation.

## Optional inventory sample points

Up to six spatially distributed points are generated within each eligible inventory polygon. Where possible, points are placed at least 20 m inside the polygon boundary and at least 60 m apart. Each point inherits the polygon assessment and is marked as `inventory_weak`, because it is not an independent field observation. Each point receives a weight of `1 / number of points in its polygon`, so the total weight of one polygon is one, regardless of the number of sampled points. These points are intended as optional additional training data, not as the main accuracy test.

## Outputs and limitations

GeoPackage files are exported for monitoring centres, 20 m monitoring circles, inventory polygons, optional sample points, and spatial blocks. CSV summaries describe fold assignments, assessment classes, observation years, and quality checks. Remote-sensing predictors are not calculated and models are not trained or evaluated at this stage. The 10 km block size and 1 km buffer are initial choices whose suitability should be checked later.

monitoring_centres – Point locations representing the centres of 20 m radius Natura forest habitat monitoring plots. Each point contains the monitoring results, including habitat type and condition assessments.

monitoring_plots_20m – Circular polygons with a 20 m radius (approximately 1,257 m²), created around the monitoring centres. They represent the area assessed during field monitoring and can be used to extract local remote-sensing characteristics.
