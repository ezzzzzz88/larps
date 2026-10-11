# Oldcircle/geo-sleuth

https://github.com/Oldcircle/geo-sleuth · checked 2026-10-11 at `3105d38`

**Verdict: not a LARP.** The code does what the README describes. The headline accuracy numbers ("±2 m", "3 m", "87 m") are the author's own runs on photos the skill was tuned on, so they can't be checked from the repo. The README says so itself.

## What it is

It's an Agent Skill (`SKILL.md`, plus about 11.5k lines of Python across 26 scripts) that guides a coding agent through photo geolocation. It uses EXIF/OCR/reverse search, OSM Overpass queries, Terrarium elevation tiles, satellite tiles, street view, CLIP/DINOv2 ranking and a candidate "board" that scores clues by likelihood ratio. The model reads the instructions and the scripts do the searching and scoring.

## What I checked

| Check | Result |
|---|---|
| Test suite (`pytest tests`) | **66 passed**, 18 subtests, 8 s, offline |
| `doctor.py --network` | No failures. Overpass, elevation, Google tiles, Street View, Yandex and Baidu image search all reachable. The browser warning came from my sandbox. |
| Links and images in the README | All present in the repo |
| `terrain.py fit` roll solving (the README's "1° tilt" story) | Real code: a least-squares solve for horizon offset plus roll (`terrain.py:955-967`), not just text in the README |
| Commit history | 22 commits over Sep 18 – Oct 10 2026 by two authors (owner plus an outside contributor who added a Sri Lanka region pack). It reads like normal, steady development. |

### My own round-trip test of the skyline matcher

1. `terrain.py view` rendered the skyline from a known point in Grindelwald, CH (46.6243, 8.0414), heading 290°, 65° HFOV.
2. `terrain.py ridge` pulled the ridge line out of that render.
3. `terrain.py fit` ran over 7 candidate areas: the true area with its center offset about 700 m, plus Lauterbrunnen, Interlaken, Meiringen, Wengen, Adelboden and a point 5 km west.

Result: the true area ranked #1 at **46.62448, 8.04146, about 20 m from the true point**, heading 289.5° (true 290°). Its error was 6.2 px; the next best area scored 15.1 px. The run took 22 s.

This test is partly circular: the "photo" and the fit use the same elevation data. It proves the pipeline is real and works end to end. It doesn't prove anything about real photos.

## Caveats and things I noticed

- **The defaults assume a camera on flat ground.** `fit` defaults to `--cam-flat 8` (at most 8 m of height difference within 300 m). In an alpine valley that filter threw out every candidate position, including the true one, and the ranking was wrong (Interlaken came first). Passing `--cam-flat 1000` fixed it. The agent has to know to loosen this filter.
- **`view` and `fit` use different camera models.** `view` renders with azimuth linear across the image (the 10° tick marks are evenly spaced), while `fit` assumes a pinhole camera. In my test the fitted focal length came out at the long end of the ladder (1800 px against a true 1607 px) and `fit` warned that it was "absorbing another error". The ranking and heading weren't affected. Don't trust `f`/`f35` from fits against `view` renders.
- **The benchmarks are self-reported, and both blind re-runs were on solved photos.** The README states that these photos were already solved and that lessons from them are baked into the skill.
- **It depends on scraping.** Reverse image search drives a headless browser against Baidu and Yandex, and the satellite imagery comes from Google's unofficial tile endpoint. These can break or get blocked at any time. That's fragile, but it isn't fake.
- **Dual-use.** It's a capable doxxing tool if pointed at private people's photos.

## Bottom line

This is real, working, unusually careful engineering. The only "marketing" is cherry-picked showcase cases, and the README is open about that.
