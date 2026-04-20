# Audit Photo Collector

**HGS Engineering, Inc.**

A mobile-first Progressive Web App (PWA) for collecting geotagged photos during environmental site audits. Designed for field use on phones and tablets — works offline after initial load.

**Repository:** [github.com/whittw1/super-guacamole](https://github.com/whittw1/super-guacamole)

---

## Features

| Feature | Description |
|---------|-------------|
| **Photo capture** | 2 default photo slots per entry + dynamic "Add Photo" for unlimited extras. Photos are resized client-side (configurable resolution & JPEG quality). |
| **GPS tracking** | Auto-captures device coordinates on each photo. Manual "GPS" button also available. Coordinates stored as latitude/longitude/accuracy per entry. |
| **Site list import** | Load a `.txt` file of site names for quick autocomplete on the Site/Session field. |
| **Save & New** | Saves the current entry and immediately resets the form for the next location. Auto-fills site name from the previous entry. |
| **Photo Log export** | Generates a formatted Word document (`.docx`) — one photo per page with sequential reference numbers (e.g., C2-001, C2-002). Includes location, details, GPS coordinates, and timestamp per photo. |
| **Backup / Import** | Export all entry metadata as JSON. Import merges or replaces existing entries. Photo binary data is excluded from backups (text-only). |
| **Offline support** | Service worker caches the app shell. HTML is network-first (always gets latest version); JS libraries are cache-first. |
| **Configurable photos** | Settings panel for resolution presets (720p / 1080p / 1440p / original / custom) and JPEG quality slider. |

## File Structure

```
index.html          Single-page app (HTML + CSS + JS, ~2200 lines)
sw.js               Service worker — cache versioning, network-first HTML strategy
manifest.json       PWA manifest (app name, theme color, display mode)
```

All application code lives in `index.html` — there is no build step. External libraries are loaded from CDN:

- **JSZip** — not currently used in export, but available
- **SheetJS (xlsx)** — available for spreadsheet operations
- **docx** — generates the Word Photo Log document

## Data Storage

| Store | Purpose |
|-------|---------|
| `localStorage` (`audit_saved`) | Serialized array of all saved entries (location, details, GPS, timestamps, photo thumbnails) |
| `localStorage` (`audit_current`) | Auto-saved state of the in-progress entry (survives page reload) |
| `localStorage` (`audit_site_list`) | Imported site names for autocomplete |
| `localStorage` (`audit_photo_settings`) | User's resolution/quality preferences |
| **IndexedDB** (`audit_photos_v1`) | Full-resolution photo binary data. Falls back to localStorage base64 if IDB unavailable. |

## Entry Data Model

Each saved entry contains:

```json
{
  "id": "e_1711234567890_a1b2",
  "siteName": "ABC Waste Services — 2026-03-15",
  "location": "Building 3 / Secondary Containment Area",
  "details": "Observed staining near drum storage",
  "latitude": 32.7767,
  "longitude": -96.7970,
  "gpsAccuracy": 12.5,
  "timestamp": "2026-03-15T14:30:00.000Z",
  "photos": {
    "p_main": { "timestamp": "...", "thumbnail": "data:image/jpeg;base64,...", "dbKey": "photo_e_..._p_main" },
    "p_wide": { "timestamp": "...", "thumbnail": "...", "dbKey": "..." }
  }
}
```

## Service Worker Strategy

- **Cache version:** `audit-collector-v2.2` (bump on each release to force refresh)
- **HTML:** Network-first — always tries to fetch latest, falls back to cache offline
- **JS/assets:** Cache-first — served from cache, falls back to network
- `self.skipWaiting()` + `clients.claim()` ensures immediate activation

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v2.2 | 2026-03-27 | Fix photo capture failing after Save & New on mobile (recreate file input on each capture) |
| v2.1 | 2026-03-26 | Add GPS coordinate tracking, rename to Audit Photo Collector |
| v2.0 | 2026-03-25 | Add Photo Log (Word doc) export with sequential reference numbering |
| v1.0 | 2026-03-24 | Initial release — photo capture, save/new, backup/import |
