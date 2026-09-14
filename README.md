# WBCrop
WBCrop: A georeferenced dataset of 18 crop types across West Bengal, India
# WBCrop — Point Dataset & Multi-Source Feature Extraction Pipeline

Crop-type classification for smallholder fields in West Bengal, India, using labelled
ground points and four satellite/embedding sources: Sentinel-1, Sentinel-2, AlphaEarth
Foundations, and Tessera.

## Dataset

**WBCrop** is a georeferenced, manually curated ground-truth dataset of 18 crop types
across West Bengal, published on Zenodo.

| | |
|---|---|
| Title | WBCrop: A georeferenced dataset of 18 crop types across West Bengal, India |
| DOI | [10.5281/zenodo.19140935](https://doi.org/10.5281/zenodo.19140935) |
| Data collector | Manas Utthasini — Dept. of Environmental Science and Engineering, IIT (ISM) Dhanbad |
| Contact | Krishnagopal Halder — Leibniz Centre for Agricultural Landscape Research (ZALF) |
| License | CC BY-NC 4.0 (non-commercial) |
| Published record | 74,351 points, 18 crop classes, 21 districts |
| Code (dataset authors) | https://github.com/geonextgis/WBCrop |

The dataset covers one full agricultural year (Jan 2023 – Dec 2024), spanning kharif,
rabi, and zaid seasons plus perennial crops. Points were collected two ways: in-situ field surveys and through remote annotation using Google Street View and
high-resolution satellite imagery, with class balance actively managed to limit long-tail
bias. Each point carries a crop label, district, approximate sowing/harvest window,
collection date, season, and phenological stage at the time of collection, in EPSG:4326.


### Point count: 74,351

The Zenodo record has 45,616 points.The working file this pipeline uses,
`wbcrop_points_extended.gpkg`, has **74,351**.

The extra points come from grid densification inside each field boundary (`00_Points_
Preparation.ipynb`): a 15 m candidate grid, at least 15 m from any original point and
5 m inside the field edge, tagged with a `source` column (`original` vs `synthetic`) so
the two are never silently mixed. This is a deliberate extension of the published release
for training-data volume, not a correction to Zenodo's number — 45,616 remains the
correct count for the dataset as record. Report model accuracy on originals-only and
on the extended set separately; densified points carry no new information, only added
weight toward large fields.

### Schema (`wbcrop_points_extended.gpkg`)

| column | type | meaning |
|---|---|---|
| `id` | int | unique point key; every notebook joins on this |
| `crop` | string | one of 18 crop classes |
| `district` | string | administrative district (21 total) |
| `state` | string | constant, `west_bengal` |
| `sowing`, `harvest` | string | approximate period, e.g. `jul_2023`; `NA` if unavailable |
| `collection_date` | string | `YYYY-MM-DD`, field survey or Street View timestamp |
| `season` | string | `kharif`, `rabi`, `zaid`, or `perennial` |
| `pheno_stage` | string | crop stage at collection (vegetative, reproductive, maturity) |
| `latitude`, `longitude` | float | WGS84 |
| `source` | string | `original` (published) or `synthetic` (densified) |
| `geometry` | point | EPSG:4326 |

## Pipeline

Run in order. `00` prepares the points; `01`–`04` extract per-source point time series;
`05`–`07` extract 128×128 patches for the same sources.

| notebook | source | output |
|---|---|---|
| `00_Points_Preparation.ipynb` | EE survey points + polygons | `01_Points/points_master.gpkg` |
| `01_S1_Point_Extraction.ipynb` | Sentinel-1 RTC (Planetary Computer) | `02_RawTimeSeries/S1/*_long.parquet` (per orbit) |
| `02_S2_Point_Extraction.ipynb` | Sentinel-2 (Earth Engine) | `02_RawTimeSeries/S2/*_long.parquet` |
| `03_AlphaEarth_Point_Extraction.ipynb` | AlphaEarth Foundations (Earth Engine) | `03_Features/AlphaEarth/*_features.parquet` |
| `04_Tessera_Point_Extraction.ipynb` | Tessera embeddings | `03_Features/Tessera/*_features.parquet` |
| `05_S1_S2_Patch_Extraction.ipynb` | Sentinel-1 + Sentinel-2 | `tile_*.zarr.zip` + `.meta.json` |
| `06_AlphaEarth_Patch_Extraction.ipynb` | AlphaEarth (EE export) | `tile_*.zarr.zip` + `.meta.json` |
| `07_Tessera_Patch_Extraction.ipynb` | Tessera (`read_patch`) | `tile_*.zarr.zip` + `.meta.json` |


### Notes for anyone running this

- **S1**: run once per orbit (`ORBIT='descending'`, then `'ascending'`); keep the two
  files separate, never merge them into one series. Do all averaging/compositing in
  linear power, not dB.
- **S2**: exported raw, no cloud mask — mask after download using `SCL`/`MSK_CLDPRB` and
  record the rule used.
- **AlphaEarth / Tessera**: both are annual embeddings (2 years / 1 year respectively for
  this AOI), not real time series. Tessera's coverage per year is measured at run time,
  not assumed from its schema list.
- **Environment split**: the S1/S2 and AlphaEarth patch notebooks require `zarr<3`; the
  Tessera patch notebook requires `zarr>=3` (a `geotessera` dependency). Run Tessera in
  its own environment — the two are not compatible in one process.

## Requirements

Google Earth Engine account/project, Microsoft Planetary Computer access (no key needed
for anonymous read), `geotessera`, and either Google Drive (Colab) or local disk with the
paths adjusted accordingly. See each notebook's setup cell for exact packages.

## License

Code in this repository: MIT.
Dataset: CC BY-NC 4.0 per the WBCrop Zenodo record — non-commercial use only, and
redistribution of the raw data should follow the terms above.

## Citation
