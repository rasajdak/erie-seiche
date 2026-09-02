# Erie Seiche

A live visualizer of Lake Erie sloshing. Fourteen water-level gauges, eight American
and six Canadian, are read every six minutes and fitted into a single lake surface,
drawn two ways: a cross-section down the lake's long axis, and a 3-D orthographic
view of the whole lake tilting.

Live at <https://ryansajdak.com/greatlakes/erie/seiche/>.

## What is in here

    index.html    the entire app: markup, CSS and JS in one file, no build step
    preview.png   Open Graph / social preview image

There are no dependencies, no package manager, no server code. The page fetches
public, keyless APIs directly from the browser.

## Running it

Any static server works. From this directory:

    python3 -m http.server 8000

then open <http://localhost:8000/>. (Opening `index.html` straight off the disk
mostly works too, but a real origin keeps the fetches predictable.)

To deploy, copy `index.html` and `preview.png` to any static host. That is the whole
deployment.

## Data sources

| Source | What it gives | Notes |
| --- | --- | --- |
| [NOAA CO-OPS](https://api.tidesandcurrents.noaa.gov/api/prod/datagetter) | US gauges, 6-minute preliminary water level, plus wind | keyless, CORS open |
| [DFO IWLS](https://api-iwls.dfo-mpo.gc.ca/api/v1/) | Canadian gauges, 3-minute water level in metres | keyless, CORS open, converted to feet |
| [Natural Earth](https://www.naturalearthdata.com/) | the lake outline, inlined as a coordinate array | public domain |

Both networks report on the same datum (Low Water Datum, IGLD 85), which is what makes
one fitted surface across the border legitimate. Everything the page draws is a
departure from each gauge's own 5-day mean, not an absolute level.

## The parts worth knowing before you edit

Line numbers are approximate and drift as you edit, but the section comments in
`index.html` are the real map.

- **`STATIONS`** (~line 260) is the whole gauge list, west to east: `{id, src, name,
  short, st, lat, lon}` where `src` is `"noaa"` or `"chs"`. `TOLEDO` and `BUFFALO` are
  just the first and last entries, and they define the lake's long axis. Everything
  downstream is projected onto that axis, so the list must stay ordered.
- **`OUTLINE`** (~line 630) is the lake shoreline as `[lon, lat]` pairs, and it drives
  both the map inset and the 3-D lattice.
- **`BOTTOM`** / **`MAX_DEPTH`** (~line 294) are a schematic depth profile along the
  axis, expressed as `[fraction of axis, feet]`. Schematic on purpose: it is a
  cross-section cartoon, not bathymetry.
- **The surface fit** (~line 392) is the mathematical heart. At each of `FITN` points
  along the axis it fits a level plus a cross-lake tilt from whatever gauges are in
  range, ridge-regularised: `FIT_CSCALE` sets the range, `FIT_RIDGE` clamps the cross
  term hard and `FIT_ARIDGE` barely touches the along-lake term. If your lake is
  gauged on one shore only, drop the cross term rather than fighting the ridge.
- **The 3-D view** (~line 752) is hand-rolled: an orthographic projection over a
  lattice, drawn to a 2-D canvas. No WebGL, no library. `yaw`/`pitch` are drag-driven.
- **Poster and movie export** (~line 1387) redraw the same scene into an off-screen
  canvas at a larger size, so anything you add to a frame needs to work at both sizes.

## Adapting it to another lake

Roughly in order of effort:

1. Replace `STATIONS` with your gauges, ordered along the axis you care about. Canadian
   station ids come from the IWLS `/stations` endpoint; US ids are CO-OPS station numbers.
2. Replace `OUTLINE` with your shoreline. Natural Earth's lakes layer, simplified to a
   couple hundred points, is what is in here.
3. Redo `BOTTOM` and `MAX_DEPTH` for the new depth profile.
4. Retune the view: `COSLAT` (~line 280) is hard-coded to Erie's latitude, and the 3-D
   camera has a default yaw chosen because Erie lies diagonally (~line 927).
5. Rewrite the prose. The header, the caption block, and the seiche history are all
   Erie-specific.

## Gotchas learned the hard way

- **Do not quantise the water colour.** An earlier version binned the tint into steps
  to save work per frame and the lake developed visible contour bands. Colour is
  computed exactly for every cell, every frame, and it is fast enough.
- **The plane fit has to be weighted and centred** on the gauges actually in range at
  that point, or the cross term runs away at the ends of the lake where only one shore
  reports.
- **Both APIs drop requests.** The loader tolerates gaps rather than failing the frame.
- **CO-OPS misfiles some gauges.** The Niagara River stations are filed under Erie in
  the CO-OPS station list even though they are not on the lake. Check what a station
  actually is before trusting its grouping.
- Gauge labels do not fit on two rows once you pass about ten stations. There is a
  three-row layout in `ROWY` (~line 564) for that reason.

## Licence and reuse

Written by Ryan Sajdak. The underlying data is public (NOAA, DFO, Natural Earth).
Fork it, strip the branding, point it at your own lake.
