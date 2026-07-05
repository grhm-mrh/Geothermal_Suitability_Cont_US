# Geothermal_Suitability_Cont_US

# Detailed Technical Workflow

# Phase 1: Conceptualization & Data Sourcing
  ## The Idea:
  To build an automated, AI-driven Multi-Criteria Decision Analysis (MCDA) model to identify highly viable geothermal drilling sites.
  The score relies on subsurface heat (Temperature-at-Depth), natural permeability (Distance to Quaternary Faults), thermal sources (Distance to Active Volcanoes), and legal viability (Protected Lands exclusion).

  # The Challenge: 
  High-resolution national temperature data was taken offline. The only available data were legacy, flat JPEG maps with heavy JPEG compression, anti-aliasing artifacts, and obscuring text/logos.

# Phase 2: Data Fixing
  # Georeferencing & Projection: 
  Imported the legacy SMU Geothermal JPEG. Discovered a projection mismatch (Conic map on a WGS84 flat grid). Corrected the workspace and reprojected to EPSG:5070 (NAD83 / Conus Albers) to guarantee mathematically           accurate Euclidean distance calculations in meters.
  # Bounding Box Fix: 
  Created a "Single parts" polygon mask of just the Lower 48 states. Clipped the raw map using this mask while explicitly setting the NoData value to 0, successfully stripping away the oceans, Canada, and the mathematical   bounding box trap.

# Phase 3: Color Quantization & Anomaly Patching
  # Unsupervised Clustering: 
  Ran RGB to PCT (Color Quantization) limited to 20 colors to translate the millions of RGB pixels into discrete IDs.
  # Mathematical Limit: 
  Discovered that unsupervised clustering deleted physically small micro-anomalies (like the SoCal and SW Idaho hotspots) to save memory for broader regions.
  # Vector Burning (Anomaly Patching):
  Bypassed the algorithmic failure by manually digitizing the missing micro-anomalies into a new vector shapefile. Rasterized these polygons (forcing the exact pixel resolution of the base map and initializing with 0) and   used Raster Calculator logic to perfectly "stamp" these anomalies back onto the base map.

# Phase 4: Reclassification & Noise Eradication
  # Cheat Sheet: 
  Used the map legend to map the random PCT ID numbers to their actual real-world temperatures (e.g., ID 17 = 150°C).
  # Noise Vaporization:
  Ran Reclassify by Table using only the verified IDs. By enabling the "Use NoData for unlisted values" function, instantly vaporized all state borders, black text, and JPEG compression noise into transparent holes.
  Morphological Fill: Ran Fill NoData with a 500-pixel maximum distance. The IDW (Inverse Distance Weighting) interpolation successfully bridged the massive gaps left by the deleted logos, restoring the map to a             continuous geological surface.
  # Decimal Snapping:
  The Fill tool generated intermediate floating-point decimals (e.g., 87.43°C). Ran a final Reclassify by Table using Range Boundaries (min <= value < max) to mathematically snap all decimals back to the 7 discrete          temperature integers, creating a flawless, 100% solid base raster.

# Phase 5: Multi-Criteria Suitability Modeling
  # Permeability & Heat Sources:
  Downloaded USGS Quaternary Faults and Holocene Volcanoes. Reprojected them to EPSG:5070, rasterized them to match the base map grid, and ran Proximity (Raster Distance) to generate continuous distance-in-meters grids.
  # Legal Exclusion Mask: 
  Rasterized a National Parks/Protected Lands shapefile into a Boolean mask (0 = Illegal to drill, 1 = Legal to drill).
  # The Final Matrix Math: 
  Executed the final MCDA formula in the Raster Calculator:
  ("Temperature" + ("Fault_Distance" <= 15000) * 50 + ("Volcano_Distance" <= 50000) * 50) * "Legal_Mask"
  # The Result: 
  Optimized, legally compliant, scored Geothermal Suitability Map.
