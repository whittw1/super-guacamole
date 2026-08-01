# Audit Photo Collector

**HGS Engineering, Inc.**

A mobile-first Progressive Web App (PWA) for collecting geotagged photos during environmental site audits. Designed for field use on phones and tablets — works offline after initial load.

**Repository:** [github.com/whittw1/super-guacamole](https://github.com/whittw1/super-guacamole)

---

## Features

| Feature | Description |
|---------|-------------|
| **Photo capture** | 2 default photo slots per entry + dynamic "Add Photo" for unlimited extras. Photos are resized client-side (configurable resolution & JPEG quality). |
| **GPS tracking** | Auto-captures device coordinates on each photo. Manual "GPS" button also available. Coordinates stored as latitude/longitude/accuracy per entry. An external Bluetooth GPS receiver improves accuracy automatically (it feeds iOS Location Services; no app changes involved). |
| **Compass facing per photo** | Magnetometer heading sampled when a photo slot is tapped (*point the device, then tap* — the page is suspended while the iOS camera is open). Landscape-corrected, converted magnetic→true north via an embedded CONUS declination grid using the entry's GPS. Reported as 8-point cardinal + degrees; readings the compass flags as worse than ±50° are suppressed, ±30–50° marked "~". Requires a one-time iOS motion-permission tap. |
| **Site association** | Every entry carries a Site/Session name; saving without one prompts for confirmation. The saved list shows each entry's site (amber "No site" warning when missing). |
| **Site & date filtering** | Filter the saved list and every export by site and date. Batch operations on the filtered set: **Set Site** (bulk-assign a site name) and **Delete** (removes entries *and* their stored photos). |
| **Site list import** | Load a `.txt` file of site names for quick autocomplete on the Site/Session field. |
| **Save & New** | Saves the current entry and immediately resets the form for the next location. Auto-fills site name from the previous entry. |
| **Readable previews** | 240 px stored thumbnails; form slots hydrate to the full-resolution photo from IndexedDB. Tapping a **filled form slot** or a saved-list thumbnail opens a full-size viewer with the facing caption — the form-slot viewer includes a Retake button (empty slots capture on tap). Old low-res thumbnails regenerate automatically on launch. |
| **ZIP export** | Photos renamed with sequential reference numbers plus the site name (`P0001_Site_Name.jpg`), bundled with an XLSX report and CSV (including a **Photo Facing** column). ZIP/report/CSV filenames carry the site when a single site is exported. |
| **Photo Log export** | Formatted Word document (`.docx`) — photo cards with sequential reference numbers, location, details, "Site:" line, GPS coordinates, and per-photo facing direction. Photo+caption pairs are marked `cantSplit` so captions never break onto the next page. Banner/header/filename show the site when the export covers one site. |
| **Export log** | Every ZIP / Photo Log export is recorded (timestamp, type, site/date filter, entry & photo counts, reference range, filename) — viewable from the Export dialog (📜) or the phone overflow menu, capped at 200 records. |
| **Backup / Import** | Export all entry metadata as JSON. Import merges or replaces existing entries. Photo binary data is excluded from backups (text-only). |
| **Offline support** | Service worker caches the app shell. HTML is network-first (always gets latest version); JS libraries are cache-first. |
| **Configurable photos** | Settings panel for resolution presets (720p / 1080p / 1440p / original / custom) and JPEG quality slider. |

> The satellite **Site Map** feature (Esri World Imagery, address search, JPG/Word export) lived here briefly (v2.5–v2.7) and was **moved to the DLA Site Notes app**, whose Address field is the natural centering input. Port instructions: `dla-site-notes/SITE-MAP-INSTRUCTIONS.md`.

## File Structure

```
index.html                  Single-page app (HTML + CSS + JS, ~3,000 lines)
sw.js                       Service worker — cache versioning, network-first HTML strategy
manifest.json               PWA manifest (app name, theme color, display mode)
staticwebapp.config.json    Azure SWA config — no-cache headers for index/sw, /notes.html redirect
Status.md                   Project status for the Claude Apps status board
.github/workflows/          GitHub Actions → Azure Static Web Apps deployment
```

All application code lives in `index.html` — there is no build step. External libraries load from CDN (first load requires connectivity):

- **JSZip** — builds the ZIP export
- **SheetJS (xlsx)** — builds the XLSX report inside the ZIP
- **docx** — generates the Word Photo Log document

## Data Storage

| Store | Purpose |
|-------|---------|
| `localStorage` (`audit_saved`) | Serialized array of all saved entries (site, location, details, GPS, timestamps, photo thumbnails + headings) |
| `localStorage` (`audit_current`) | Auto-saved state of the in-progress entry (survives page reload) |
| `localStorage` (`audit_site_list`) | Imported site names for autocomplete |
| `localStorage` (`audit_photo_settings`) | User's resolution/quality preferences |
| `localStorage` (`audit_export_log`) | Export history (newest first, max 200 records) |
| **IndexedDB** (`audit_photos_v1`) | Full-resolution photo binary data, keyed `<entryId>__<slotId>`. Falls back to localStorage base64 if IDB unavailable. Deleting entries (single or batch) also deletes their photos here. |

Startup housekeeping: regenerates pre-v2.4 low-res thumbnails from stored photos, and purges cached site-map data left behind by the retired v2.5–v2.7 feature.

## Entry Data Model

Each saved entry contains:

```json
{
  "id": "e_1711234567890_a1b2",
  "siteName": "ABC Waste Services",
  "location": "Building 3 / Secondary Containment Area",
  "details": "Observed staining near drum storage",
  "latitude": 32.7767,
  "longitude": -96.7970,
  "gpsAccuracy": 12.5,
  "timestamp": "2026-03-15T14:30:00.000Z",
  "photos": {
    "p_main": {
      "timestamp": "...",
      "thumbnail": "data:image/jpeg;base64,...",
      "thumbV": 2,
      "fileType": "image/jpeg",
      "dbKey": "e_1711234567890_a1b2__p_main",
      "hdgMag": 30,
      "hdgAcc": 12
    },
    "p_wide": { "...": "..." },
    "p_extra_0": { "...": "..." }
  }
}
```

- `thumbV` — thumbnail format version; entries below the current version get their thumbnails regenerated from IndexedDB on launch.
- `hdgMag` / `hdgAcc` — magnetic compass heading (degrees) and compass-reported accuracy, sampled at photo-slot tap time. Optional; absent when permission was denied or no reading was fresh. True-north conversion happens at display/export time using the entry's coordinates (`declinationAt` bilinear grid), so headings render "(mag)" if the entry has no GPS.

## Service Worker Strategy

- **Cache version:** `audit-collector-v3.7` (bump on every `index.html` release to force refresh)
- **HTML:** Network-first — always tries to fetch latest, falls back to cache offline
- **JS/assets:** Cache-first — served from cache, falls back to network
- `self.skipWaiting()` + `clients.claim()` ensures immediate activation

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v2.10 | 2026-07-31 | Tap a filled photo slot to view the photo full size (verification in the field); Retake moved into the viewer |
| v2.9 | 2026-07-31 | Compass facing direction per photo (magnetometer at slot-tap, true-north conversion, cardinal labels, low-confidence suppression); Photo Facing column in CSV/XLSX |
| v2.8 | 2026-07-31 | Satellite Site Map moved out to the DLA Site Notes app (port spec in that repo); export-log history for old map exports still renders |
| v2.7 | 2026-07-31 | Site map no longer embedded in the Photo Log — separate JPG/Word deliverables only |
| v2.6 | 2026-07-31 | Removed entry-location markers from the site map (client preference; GPS not accurate enough) |
| v2.5 | 2026-07-30 | Satellite Site Map export (Esri World Imagery, address search, drag-to-pan, JPG + standalone Word doc); Photo Log fix: photo+caption rows marked `cantSplit` so captions stay with their photos |
| v2.4 | 2026-07-30 | Readable thumbnails (240 px), full-res slot previews, tap-to-view photo viewer, automatic thumbnail regeneration |
| v2.3 | 2026-07-30 | Site/date filtering, batch Set Site / Delete, site-tagged export filenames & captions, export log, no-site save warning, entry deletion now removes IndexedDB photos |
| v2.2 | 2026-03-27 | Fix photo capture failing after Save & New on mobile (recreate file input on each capture); later visual restyle to HGS Portal / DLA palette |
| v2.1 | 2026-03-26 | Add GPS coordinate tracking, rename to Audit Photo Collector |
| v2.0 | 2026-03-25 | Add Photo Log (Word doc) export with sequential reference numbering |
| v1.0 | 2026-03-24 | Initial release — photo capture, save/new, backup/import |
