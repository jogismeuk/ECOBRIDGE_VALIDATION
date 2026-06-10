# ECOBRIDGE Validation

This repository contains validation scripts for ECOBRIDGE raster datasets, specifically designed to process, combine, and validate land use and land cover (LULC) transitions across multiple scenarios and timeframes.

## Overview

The project manages geospatial raster data at multiple resolutions (1km and 10m) and validates transitions against a universal reference table to ensure data integrity and consistency.

## Scripts

### 1. **Combinator**
Combines multiple raster datasets into unified output rasters for each scenario and year.

**Purpose:**
- Merges baseline rasters (1km and 10m resolution) with scenario-specific path rasters and validation rasters
- Creates combined datasets suitable for transition validation

**Key Features:**
- Processes multiple scenarios (e.g., P4) across multiple years (2030, 2042, 2050)
- Uses ArcGIS Spatial Analyst `Combine` function to merge input rasters
- Includes progress tracking with optional `tqdm` library
- Input validation with informative warnings for missing datasets
- Outputs to `Combined_Datasets.gdb`

**Input Rasters:**
- `Baseline20248bit` (1km resolution baseline)
- `B_LULC_10m` (10m resolution baseline)
- `Path{N}_{YEAR}` (scenario-specific path rasters from main geodatabase)
- `{SCENARIO}_{YEAR}_RAW` (validation rasters from validation geodatabase)

**Output:**
- `Comb_{SCENARIO}_{YEAR}` combined rasters in `Combined_Datasets.gdb`

---

### 2. **Check_Combs_and_TT**
Validates raster transitions against a universal transition table to ensure compliance with defined rules.

**Purpose:**
- Verifies that all raster value transitions follow established rules
- Identifies invalid transitions and reports discrepancies
- Supports quality assurance and error detection in LULC datasets

**Validation Logic:**

The script enforces two key rules:

1. **Low Resolution (LR) Changes Must Match High Resolution (HR) Changes:**
   - If `initial_tile_class == new_tile_class` (LR unchanged)
   - Then `initial_cell_class MUST == new_cell_class` (HR must also be unchanged)
   - Violations flagged as "HR changed while LR did not"

2. **Tile-Cell Class Transitions Must Be Authorized:**
   - If `initial_tile_class != new_tile_class` (LR changes)
   - AND `initial_cell_class != new_cell_class` (HR also changes)
   - Then the combo `(initial_tile_class, new_tile_class, initial_cell_class, new_cell_class)` MUST exist in the universal transition table
   - Violations flagged as "Tile+cell change not found in universal table"

**Input Data:**
- Combined rasters matching pattern `Combine*v2` from `Combined_Datasets.gdb`
- Universal transition table: `DESNZ_TT_8Bits_ExportTable` containing valid transitions

**Output:**
- Console report showing:
  - ✅ Valid datasets
  - ❌ Invalid transitions with error counts and examples (first 5 violations shown)

---

## Data Structure

### Geodatabases

| Database | Location | Purpose |
|----------|----------|---------|
| `DecemberDelivery.gdb` | `C:\T6DinB\Downscaler\` | Main baseline and scenario rasters |
| `NewDownscaler.gdb` | `C:\T6DinB\Downscaler\` | Validation/downscaled rasters |
| `Combined_Datasets.gdb` | `C:\T6DinB\Downscaler\` | Output combined rasters and universal table |

### Raster Fields (Validation)

The validation script reads the following fields (indices 3–6) from combined rasters:

1. **initial_tile_class** - Initial low-resolution classification
2. **new_tile_class** - New low-resolution classification
3. **initial_cell_class** - Initial high-resolution classification
4. **new_cell_class** - New high-resolution classification

---

## Usage

### Running the Combinator

```bash
python Combinator
```

**Prerequisites:**
- ArcGIS Pro with Spatial Analyst extension
- Input rasters must exist in specified geodatabases
- Write permissions to `Combined_Datasets.gdb`

**Optional:**
- Install `tqdm` for progress bars: `pip install tqdm`

### Running the Validation Check

```bash
python Check_Combs_and_TT
```

**Prerequisites:**
- Combined rasters must exist in `Combined_Datasets.gdb`
- Universal transition table must be available
- ArcGIS Spatial Analyst extension

**Output Interpretation:**
- **✅ All transitions valid** - No errors detected
- **❌ HR changed while LR did not** - Rule 1 violation (see validation logic)
- **❌ Tile+cell change not found in universal table** - Rule 2 violation (see validation logic)

---

## Requirements

- Python 3.6+
- ArcGIS Pro with Spatial Analyst extension
- `arcpy` library (included with ArcGIS Pro)
- `tqdm` (optional, for progress bars)

---

## Error Handling

Both scripts include:
- Input existence validation with skip-on-error behavior
- Normalization of float values to integers when appropriate
- Informative console messages with emoji indicators
- ArcGIS message logging via `arcpy.AddMessage()`

---

## Notes

- Scripts use 8-bit raster values for efficient storage and processing
- Scenarios are configured as P4; expand the `scenarios` list to process additional scenarios
- The universal transition table (`DESNZ_TT_8Bits_ExportTable`) serves as the authoritative reference for valid transitions
- Validation checks are case-sensitive for string values; numeric values are normalized

---

## Recent Updates

- **June 10, 2026:** Added Combinator script for multi-resolution raster combining
- **June 10, 2026:** Added Check_Combs_and_TT validation script with transition rule enforcement

---

## Author

Josep Serra (jogismeuk)

---

## License

[Add your license here]
