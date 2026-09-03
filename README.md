# Peak Finder: can free satellite imagery track Wrightsville Beach sandbars?

**Status: closed, negative result.** September 2026.

The question was whether free Sentinel-2 imagery could show where breaking has concentrated along a 1 km stretch of Wrightsville Beach, NC, well enough to tell a surfer where to paddle out. One session of pulling real imagery answered it. It cannot, and the reasons are structural, not fixable with better code.

## The three findings

### 1. Too few scenes ever catch the outer bar breaking

Wrightsville has two bars. At low tide both break, outside and inside, even on small days. At high tide the inside breaks and the outer bar only shows on large days. The outer bar is the one that matters, so a scene is only useful near low tide with some swell, or at high tide with real size. Sentinel-2 passes at a fixed time, about 11:48 am local, so the tide stage at overpass drifts through the lunar cycle, and the size that shows the outer bar at high tide arrives with the clouds.

Joined against NOAA tide predictions (station 8658163) and Open-Meteo marine hindcast, the 18 months of low-cloud scenes over the pilot box break down like this:

| Filter on scenes with under 30% tile cloud, Mar 2025 to Sep 2026 | Count |
|---|---|
| Low-cloud scene dates | 75 |
| Within 2 hours of low tide at overpass | 24 |
| Of those, wave height at least 0.5 m (a small but real day) | 13 |
| Of those, wave height at least 0.8 m | 2 |
| Within 2 hours of high tide at overpass | 24 |
| Of those, wave height at least 1.0 m (large enough to break outside) | 1 |
| Usable under either rule | 14 |

Fourteen candidate scenes in 18 months, under one a month, and eleven of them on days of 0.5 to 0.7 m where the outer line is thin. The pilot's own kill threshold was fewer than about one usable scene a month. Full inventory in `scenes_wb_2025-03_to_2026-09.csv`, with the rule applied per scene.

Tile-level cloud cover is also a poor filter: 7 of the 16 most recent "clean" tiles had clouds sitting over the island (image 01).

### 2. The foam signal is the inner bar, not the outer bar

At low tide the inside and outside both break, and the inside throws far more foam. The one dead-low-tide scene in the summer batch (June 13, 2026) showed a broad bright band along the whole beach with no separable outer line: shorebreak and inner-bar foam at low water, merging with whatever the outside was doing. A foam index at 10 m would mostly measure how foamy the inside is (image 02). Cross-shore masking could separate the two, since the outer bar sits 10 to 20 pixels offshore, but that fixes contamination, not the scene count.

### 3. Ten-meter pixels cannot resolve a peak even at a world-class point

As a truth check, the same pull was run over the Superbank (Snapper Rocks to Kirra, sand bottom) and Jeffreys Bay (Supertubes) for the biggest clear days of the last year, up to 2.4 m at 12 seconds with zero cloud (images 05 to 08).

The best case, Superbank on March 25, 2026 at 2.0 m, shows the surf zone as a continuous white band 3 to 6 pixels wide with a hint of two or three parallel foam lines near Snapper. J-Bay on its biggest day is one thick band with a bulge at the point. The satellite never resolves a wave, a section, or a takeoff spot. Its ceiling is "the surf zone was wide and bright at this alongshore position," at roughly 50 to 100 m precision. Compositing averages bands into bands.

The Queensland government's free 10 cm aerial of the same footprint (image 09, 10) shows what resolution buys: individual waves peeling down the point, the sandbank visible through the water, the channel outside it. That frame was flown once, in mid 2022.

## Other free sources checked

| Source | Resolution | Cadence over the box | Verdict |
|---|---|---|---|
| Sentinel-2 L2A (AWS Earth Search, CDSE) | 10 m | about 4 low-cloud dates a month, 3 tiles overlap the box | Primary source. Too coarse, wrong timing. |
| Sentinel-1 radar (relative orbit 77, ascending) | 10 m pixels, about 20 by 22 m effective, plus speckle | every 12 days, one satellite, about 7:13 pm local | Sees through cloud, shows surf zone as a faint fringe. Wind-dependent. Does not rescue resolution (image 04). |
| NAIP aerial (USDA) | 0.6 to 1 m | 2012, 2014, 2016, 2018, 2020, 2022 | Beautiful snapshots of bar structure (image 03). Years apart. |
| NC OneMap orthoimagery | 15 to 30 cm | about every 4 years | Same. |
| NOAA emergency response imagery | 15 to 50 cm | after hurricanes only | Same, but timed right after the swells that rebuild bars. |
| Landsat 8/9 | 30 m | frequent | Worse than Sentinel-2. |
| Maxar Open Data | 30 cm | disaster releases only | No Wrightsville or Gold Coast event in the catalog. |
| Google Earth historical imagery | 30 to 50 cm | dozens of dates | View only. Best tool for checking session memory against past bar states. |
| PlanetScope / SkySat | 3 m daily / 50 cm tasked | commercial | Not free. |

## What would actually work

A fixed camera doing ten-minute time exposures at low tide, the Argus method used at the Duck, NC field research facility. It paints the outer bar and its rip gaps at meter scale, every low tide, in any light. Wrightsville already has public webcams at Johnnie Mercer's Pier, Crystal Pier and the Blockade Runner. The first step, if this is ever picked up again, is a screenshot of each to see which one covers the Access 35 to Hanover Seaside Club stretch, then a conversation with the operators before anything automated. Everyone already knows where the breaks are. The open question is only which one is on this week.

## Reproducibility

Everything here ran from a cloud container over plane WiFi in one session, with no data downloaded locally beyond the crops.

- Scene search: STAC POST to `https://earth-search.aws.element84.com/v1/search`, collection `sentinel-2-l2a`, bbox `[-77.800, 34.190, -77.788, 34.212]`, `eo:cloud_cover < 30`. CDSE STAC search works unauthenticated too.
- Crops: rasterio windowed reads of the AWS Sentinel-2 COGs (`TCI.tif`), about one second per scene for the island. No full-tile downloads.
- Radar: Microsoft Planetary Computer `sentinel-1-rtc`, VV band, SAS token from the public token endpoint.
- NAIP: Planetary Computer `naip` collection, same token pattern.
- Tides: NOAA CO-OPS predictions API, station 8658163, MLLW, GMT.
- Swell: Open-Meteo marine API, hourly wave height and period at 34.20, -77.79.
- Queensland aerial: public ArcGIS ImageServer `Basemaps/LatestStateProgram_AllUsers`, exportImage with a WGS84 bbox.

## Images

1. `img/01_wb_island_16_dates.jpg`: whole island, 16 most recent low-cloud dates, true color.
2. `img/02_wb_pilot_zoom_9_clean_dates.jpg`: pilot stretch, 9 clean dates, raw and contrast-stretched.
3. `img/03_wb_sentinel2_vs_naip.jpg`: same footprint, Sentinel-2 10 m beside NAIP 60 cm.
4. `img/04_wb_sentinel1_radar_vs_optical.jpg`: Sentinel-1 radar passes beside the June 3 optical scene.
5. `img/05_superbank_sentinel2_big_days.jpg`: Superbank, biggest clear days.
6. `img/06_jbay_sentinel2_big_days.jpg`: J-Bay, biggest clear days.
7. `img/07_superbank_native_pixels_x6.jpg`: Snapper Rocks native pixels at 6x.
8. `img/08_jbay_native_pixels_x6.jpg`: Supertubes native pixels at 6x.
9. `img/09_superbank_sentinel2_vs_10cm_aerial.jpg`: same footprint, Sentinel-2 beside 10 cm aerial.
10. `img/10_superbank_snapper_10cm_aerial.jpg`: Snapper Rocks to Greenmount at 10 cm, mid 2022.

Imagery credits: Copernicus Sentinel data 2025 to 2026; USDA NAIP; State of Queensland aerial imagery, reproduced under its open licence.
