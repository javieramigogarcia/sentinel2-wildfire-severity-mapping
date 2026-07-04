# Sentinel-2 Wildfire Severity Mapping

Change detection and burn severity mapping using Sentinel-2 satellite imagery and Google Earth Engine, applied to one of the wildfires from the August 2025 Ourense–Lugo–León fire complex in northwestern Spain.

## Context

In August 2025, Spain experienced its worst wildfire season since 1994, with several large fires burning simultaneously across Galicia and León. According to press reports, the combined complex around Larouco, Quiroga, and surrounding areas burned an estimated **37,765 hectares**.

This project focuses on one of the individual fires within that complex, officially recorded by the [European Forest Fire Information System (EFFIS)](https://effis.jrc.ec.europa.eu/):

| Field | Value |
|---|---|
| EFFIS Fire ID | 279404 |
| Detected | 2025-08-10 |
| Location | 42.8847°N, -6.5724°W (near Anllares, León) |
| Official area (EFFIS) | 7,821 ha |

## Objective

Estimate the burned area and vegetation loss caused by the fire using freely available Sentinel-2 imagery, and compare the result against publicly reported figures, without downloading any imagery locally.

## Methodology

All processing was done in the cloud through the Google Earth Engine (GEE) Python API, avoiding local downloads of Sentinel-2 `.SAFE` files.

1. **Area of interest**: a circular buffer (20 km radius) centered on the fire location.
2. **Imagery source**: `COPERNICUS/S2_SR_HARMONIZED` (Sentinel-2 Surface Reflectance).
3. **Cloud masking**: pixel-level masking using the Scene Classification Layer (SCL band), removing clouds, cloud shadows, and cirrus.
4. **Composites**: a median composite was built for two time windows:
   - Pre-fire: August 1–14, 2025
   - Post-fire: September 1–15, 2025
5. **Vegetation indices**:
   - NDVI (Normalized Difference Vegetation Index), using bands B8 and B4
   - NBR (Normalized Burn Ratio), using bands B8 and B12
6. **Change detection**: pixel-wise difference between pre-fire and post-fire indices (dNDVI and dNBR). A positive value indicates vegetation loss.
7. **Burned area estimate**: pixels with dNBR greater than 0.2 were classified as affected, and their total area was calculated in hectares.

## Results

| Metric | Value |
|---|---|
| Estimated burned area (this study) | 36,477.79 ha |
| Reported complex-wide area (press) | 37,765 ha |
| Difference | ~3.4% |

The map below shows the dNDVI/dNBR change layer over the area of interest. Red and purple tones indicate the strongest vegetation loss, closely matching the shape of the fire scar visible in satellite imagery.

![dNDVI and dNBR change map](images/map_dNDVI_dNBR.png)


## Limitations

- The area of interest is a circular buffer around the fire's central point, not the exact official fire perimeter. This was a deliberate simplification given the project's one-day timeframe.
- The comparison figure of 37,765 ha refers to the entire multi-fire complex reported in the press, not to a single official perimeter matching this specific area of interest. The result should be read as an approximate, order-of-magnitude validation rather than an exact match.
- Burn severity thresholds (dNBR > 0.2) were set using a single cut-off value rather than the full USGS severity classification scale, so the result does not distinguish between low, moderate, and high severity areas.
- Cloud masking relies on Sentinel-2's SCL band, which can occasionally misclassify smoke as cloud, slightly affecting the quality of the composites closest to the fire dates.

## Tech stack

- Python 3.11
- Google Earth Engine (`earthengine-api`)
- `geemap` for interactive visualization
- Jupyter Lab

## Repository structure

```
sentinel2-wildfire-severity-mapping/
├── README.md
├── notebooks/
│   └── sentinel2_wildfire_analysis.ipynb
├── images/
│   └── map_dNDVI_dNBR.png
└── environment.yml
```

## Data sources

- Sentinel-2 imagery: Copernicus Sentinel-2 mission (ESA), accessed via Google Earth Engine
- Fire event data: [EFFIS - European Forest Fire Information System](https://effis.jrc.ec.europa.eu/)
