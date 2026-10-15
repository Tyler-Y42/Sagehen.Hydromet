# Sagehen Hydromet: data source research

Done 2026-10-14 (overnight). Every source below was probed directly; "CORS" means a browser page on another site can read it.

## The watershed

- Sagehen Creek above USGS 10343500, from the USGS NLDI basin service: 27.7 km², outlet at 39.4317, -120.2369, about 6,330 ft. The west edge runs along the crest near Independence Lake (about 8,400 ft).
- Inside or on the boundary: USGS 10343500 (outlet), Independence Lake SNOTEL 541 (west ridge, 8,340 ft), the field station's East Meadow tower and three PhenoCams (all within 400 m of the outlet). All other stations are outside.

## Used live in the page (browser only, no GitHub Action needed)

| Source | What | Where | Operator | Cadence, latency | Access | Notes |
|---|---|---|---|---|---|---|
| USGS 10343500 Sagehen Creek near Truckee | Discharge, gage height, water temperature (conductance listed but empty) | Basin outlet, 6,331 ft | USGS (Hydrologic Benchmark Network, record since 1953) | 15 min, under 1 h | NWIS IV JSON, CORS yes, no key | Primary flow. Provisional. |
| Field station East Meadow tower (Dendra) | Geonor weighing precip gauge (raw vibrating wire frequency), air temp and RH at 25 and 100 ft, wind at 25 and 100 ft, solar, pressure, leaf wetness | 39.4325, -120.2369, 6,337 ft, beside the outlet | UC Natural Reserve System climate network (Dendra, api.dendra.science), station also run with DRI/WRCC | 10 min, about 10 to 20 min | Dendra REST JSON, CORS yes, no key | The published mm and cm precip streams stopped in 2020 and 2018, but the raw frequency is still live. See Geonor below. Snow depth stream dead since 2025-08-28. |
| Synoptic SGHEN | Same tower's met variables (no precip) | as above | DRI via Synoptic | 10 min | Synoptic token in keys.js | Used for temperature and the rain line. |
| NRCS SNOTEL 541 Independence Lake | SWE, depth, precip (0.1 in steps), temp | 39.4275, -120.3134, 8,340 ft, in basin on the west ridge | NRCS (CDEC copy: IDP) | hourly, about 40 min | AWDB REST (CORS yes) and Synoptic ILKC1 | Has a daily median. The handoff's "SNOTEL 540" is Independence Creek; Independence Lake is 541. |
| SNOTEL 539 Independence Camp | as above | 6,970 ft, 3 mi NW | NRCS | hourly | AWDB, Synoptic ICPC1 | |
| SNOTEL 540 Independence Creek | as above | 6,440 ft, 4.6 mi N | NRCS | hourly | AWDB, Synoptic ICKC1 | |
| SNOTEL 428 CSS Lab | as above | 6,890 ft, Donner Summit, 10 mi SW | NRCS with UC Berkeley Central Sierra Snow Lab | hourly | AWDB, Synoptic CSSC1 | Context. |
| SNOTEL 834 Truckee #2 | as above | 6,500 ft, 9.5 mi S | NRCS | hourly | Synoptic TRUC1 | Context. |
| RAWS Stampede SMDC1, Rice Canyon RNYC1, Tahoe Donner TADC1, Dog Valley DOGC1 | precip (unheated tipping bucket), temp, wind; TADC1 also snow depth and soil | 6,000 to 7,400 ft | USFS/BLM RAWS | hourly | Synoptic | Context; buckets miss snow. |
| NV Energy NV012 Hobart Mills | 10 min precip, temp | 6,030 ft, 3 mi SE | NV Energy | 10 min | Synoptic | Nearest other gauge. |
| IEM IEMRE hourly | NCEP Stage IV radar plus gauge precip at a point | basin centroid | Iowa Environmental Mesonet | hourly, 2 to 3 h | JSON, CORS yes | 12/21/2025: 74.9 mm vs 100 mm at the Geonor. |
| IEM NEXRAD mosaic tiles | reflectivity | anywhere | IEM | 5 min | PNG tiles, CORS yes | Decoded in the browser for "radar now"; map overlay. Reno (KRGX) and Sacramento (KDAX) radars see the basin over the crest. |
| NWS gridded forecast | precip chance, amount, temperature, snow level | REV 28,105 (basin) | NWS Reno | hourly issue | api.weather.gov, CORS yes | Snow level is a forecast field here, useful next to the measured rain line. |
| PhenoCam sagehen, sagehen2, sagehen3 | still images: above canopy across the basin, the meadow fen, and a third view | at the flux tower, 6,350 ft | Kirchner lab (ETH Zurich) via the PhenoCam Network | every 30 min in daylight, upload about 1 h | phenocam.nau.edu/data/latest/{site}.jpg | Image tags need no CORS. |

## Found but not used live

| Source | Why not |
|---|---|
| CDEC SGH (NWS cooperative daily precip at Sagehen, 6,337 ft) | Daily only, missing since 2026-09-11, and CDEC sends no CORS header (would need the Action route). 12/21/2025: 4.06 in, the Geonor got 3.95 in. Good for a later climate check. |
| CDEC IDP, IDC (Independence Lake and Camp) | Copies of the SNOTEL sites. |
| WRCC Sagehen pages (wrcc.dri.edu/weather/sagh.html, the tower map for towers 1 to 11, data lister) | The site moved to wrcc-archive.dri.edu; the "most recent 24 h" table and the lister both return empty rows, and the tower pages point at a dead server (hungus.dri.edu). The inventory still lists data through Oct 2026, so the data exist; worth an email to WRCC. |
| Sagehen Keck HydroWatch towers (12 radio linked towers on an elevation transect) | Only through WRCC, see above. Not reachable. |
| AmeriFlux US-SHC (eddy flux tower, 2017 to present) | Annual releases, archive only. |
| NADP CA50 (precip chemistry, East Meadow) | Weekly lab data. |
| Long term micromet and hydrology dataset 1954 to 2018 (UNR ScholarWorks) | Archive, good for medians later. |
| Lower Camp NOAA COOP USC00047641 (1954 to 2006) | Archive. |
| Snowcloud wireless snow and soil network (Glaser, Bales) | Research deployment; no live feed found. |
| ALERTCalifornia / AlertTahoe fire cameras (one near the top of the watershed per the station, plus Sardine Peak lookout) | Images are behind their app; no stable public still URL found. |
| Synoptic CWOP stations (F6155, G6514, F0079) near Tahoe Donner | Amateur gauges, since midnight totals only. Left out. |

## Geonor weighing gauge: recovering a live precip record

- Dendra publishes the raw wire frequency (Hz) every 10 min. From Oct 2017 to Jul 2020 it also published bucket content in mm. Over 135,169 matched pairs the mm value is exactly a quadratic of frequency: content = 9.0948e-5 f² − 0.028877 f − 73.951 (residual 0.09 mm, the same each year). That is the logger's own calibration, so the page applies it to today's frequency.
- QC (tested on water year 2026 against SNOTEL daily precip):
  - Content is the median of the last hour (7 readings). Precip counts when that median rises 0.25 mm above the last level.
  - A drop of more than 1 mm held for an hour (evaporation, servicing) or more than 15 mm at once (bucket drains on 10/2, 12/23, 2/17 and 7/13) resets the level without counting anything.
  - Erratic data: if readings within 30 min of each other span more than 8 mm and swing both up and down by more than 4 mm, the gauge is faulting. The Aug to Sep 2026 fault (content jumping tens of mm up and down) is caught this way; without this rule it added 485 mm of false precip in August. Runs of 2 h or more also drop the 3 h either side. In all, 6,812 of 55,330 readings in water year 2026 were set aside (missing or erratic), nearly all in Aug and Sep.
  - Frequencies outside 500 to 3,500 Hz are missing.
- Result, 3 Oct 2025 to 31 Jul 2026 (after the 10/2 drain): Geonor 889 mm, Independence Camp 790 mm, Independence Lake 1,199 mm. Two day totals correlate at r = 0.95 (Camp) and 0.98 (Lake). False precip on dry spells: 1.6 mm all year.
- Status now: back to normal since late September 2026, but it missed the 9/3 to 9/4 storm.

## Second pass, 2026-10-15: looking for more stations in the forest

- **Added: USGS 392724120171001, Independence C precip gage nr Truckee.** 39.4566, -120.2871, about 6,920 ft (beside Independence Camp SNOTEL, 3 mi NW of the basin). 15 min precipitation totals via NWIS IV (CORS yes). Not on MesoWest, so the first sweep missed it. Dec 19 to 27, 2025: 255 mm, against 267 mm at Independence Camp SNOTEL.
- **Added: ALERTCalifornia fire camera Axis-Sagehen5 on Tower 5, top of the watershed (8,389 ft).** Public still at cameras.alertcalifornia.org/public-camera-data/Axis-Sagehen5/latest-frame.jpg, about every minute. It pans (azimuth 26° on 10/15, looking across Stampede). ALERTCalifornia publishes no coordinates for it.
- **Sagehen's own tower network (12 towers, Keck HydroWatch, later DRI snow pillows and gauges) is not public anywhere I could reach.** Dendra has "Sagehen" and "HydroWatch" organizations, but both are hidden with no public stations; the Keck sensor database (sensors.berkeley.edu) returns 503; WRCC's tower pages point at a dead DRI server. Worth asking the field station staff or DRI for access; the towers would give a real elevation transect inside the basin.
- Synoptic, all stations within 8 mi including inactive: nothing else in the basin. TS052 (TNF02 portable RAWS) sat at 39.4404, -120.2253 by the station until 2026-03-23 (fire season deployment). ICRC1 is Independence Creek stage (HADS). ILAC1 Independence Lake ended 2009.
- Other USGS sites nearby: 10343000 Independence Creek (live flow, but below the Independence Lake dam, so regulated), 10342900 Independence Lake storage (live), 10341950 Little Truckee below diversion (no current data), 10343200 Little Truckee at Hwy 89.
- Central Sierra Snow Lab stations on Dendra (Sky-To-Stream platform, Castle Creek, Onion Creek, Donner Lake West, Forest Tower): 8 to 10 mi SW at Donner Summit; most are hidden. Possible context later.
- HydroShare DWCZ Sagehen soil moisture, soil temperature and snow depth array at the Tower 1 hillslope (2017 on): file archive, last updated May 2023, not live.
- UNR Harpold lab field sites at Sagehen: the PhenoCams, the Tower 5 camera and Independence Lake SNOTEL; no separate live stations.

## PhenoCam greenness and snow series (added 2026-10-15)

- Daily summaries per camera region: sagehen_EN_1000 (canopy), sagehen2_GR_1000 and sagehen3_GR_1001 (fen meadow), from phenocam.nau.edu/data/archive/{site}/ROI/{roi}_1day.csv, records from 2017 (sagehen3 from 2019). PhenoCam processes them a few days behind (on 10/15 the latest day was 10/11).
- The archive CSVs send no CORS header (the PhenoCam API does, but has no time series). So the Alta Hydromet Action copies them: scripts/fetch_phenocam.py in Tyler-Y42/Alta.Hydromet writes data branch sagehen/phenocam.json (about 285 KB), downloading only when a file's Last-Modified changes. It rides on Alta's Cloudflare triggered 15 min job because new GitHub schedules have not been firing.
- The files' snow_flag column is empty, so snow is derived: the ROI blue chromatic coordinate b/(r+g+b), mean of the two meadow cameras, at 0.32 or more. Against Independence Creek SNOTEL depth of 1 in or more (2 mi N, 6,440 ft), 2017 to 2026: 96% agreement on 3,218 days; snow on and off dates within days most winters. Most disagreements are fall dustings and the open meadow melting out earlier or later than the SNOTEL site (WY2026: meadow bare 3/24, SNOTEL about 4/10).

## Notes for later

- SNOTEL hourly AWDB stamps are PST (same as Alta); SCAN would be different.
- IEMRE hourly is Stage IV, which uses gauges, so it is not independent of SNOTEL.
- The field station also hosts USGS water quality sampling (QW since 1968) and the NTN deposition site; neither is live.
