# Audit Photo Collector — Status

**Last updated:** 2026-07-31

## Status: ✅ Fully Functional (Deployed & In Use)

The app is stable, deployed to Azure Static Web Apps, and suitable for field use. Current version is v2.7 (2026-07-31): satellite Site Map export (Esri World Imagery, address search, drag-to-pan) delivered as a separate JPG or standalone Word document only, plus a Photo Log fix keeping captions on the same page as their photos. Per client preference, the site map has no location markers and is not embedded in the Photo Log. v2.4 added readable thumbnails; v2.3 added site/date filtering, site-tagged exports, export log, and batch operations.

## Summary

Audit Photo Collector is a mobile-first Progressive Web App built for HGS Engineering field staff to collect geotagged photos during environmental site audits. It runs entirely in the browser as a single-page app (no build step, no backend) and works offline after the first load thanks to a service worker. In the field, an auditor captures photos per location entry (two default slots plus unlimited extras), and the app automatically records GPS coordinates, timestamps, location/details text, and resizes photos client-side at a configurable resolution and quality. A "Save & New" workflow lets the auditor move quickly between locations, with site-name autocomplete from an importable site list. Back at the office, the app exports a formatted Word-document Photo Log — one photo per page with sequential reference numbers (e.g., C2-001), location, details, GPS coordinates, and timestamp — ready for inclusion in audit reports. Entries are stored locally (localStorage for metadata, IndexedDB for full-resolution photos), with a JSON backup/import feature for entry metadata. It is deployed via GitHub Actions to Azure Static Web Apps; a companion app, DLA Site Notes, was split out into its own repository, and this app redirects its old `/notes.html` path there.

## Capabilities

- Photo capture with client-side resizing (720p–1440p presets, quality slider)
- Readable previews: 240px stored thumbnails, form slots hydrate to the full-resolution photo, saved-list thumbnails are tap-to-view (full-size lightbox); old low-res thumbnails regenerate automatically from stored photos on launch
- Satellite Site Map export (Esri World Imagery tiles, no API key): auto-centers on the filtered entries' GPS points, address search (OSM Nominatim), drag-to-pan / zoom / editable coordinates, scale bar + north arrow + attribution (no location markers, per client preference), JPG download or standalone Word document (Facility Overview attachment) — kept separate from the Photo Log. Requires connectivity to generate.
- Automatic + manual GPS coordinate capture per entry
- Save & New workflow with site-name autocomplete (importable `.txt` site list); warns before saving an entry with no site name
- Edit previously saved entries; saved list shows each entry's site name (amber "No site" warning when missing)
- Site + date filters on the saved list and in the Export dialog (both ZIP and Photo Log exports respect them)
- Batch operations on the filtered set: **Set Site** (assign a site name to many entries at once) and **Delete** (removes entries and their stored photos)
- ZIP export: photo files named `P0001_Site_Name.jpg`; ZIP/report/CSV filenames carry the site name when a single site is exported
- Word (.docx) Photo Log export with sequential photo reference numbers; captions include a "Site:" line, and the banner/header/filename show the site when the export covers one site
- Export log: every ZIP / Photo Log export recorded (when, filters, entry/photo counts, ref range, filename), viewable from the Export dialog or the phone overflow menu
- JSON backup / import (metadata only; photos excluded)
- Offline support via service worker (network-first HTML, cache-first libraries)
- PWA installable on phones/tablets

## Known Limitations

- Photo binary data is not included in JSON backups — a device loss means photo loss unless the Photo Log was exported
- All data is device-local; no sync or cloud storage between devices/users
- External JS libraries (docx, SheetJS, JSZip) load from CDN, so the very first load requires connectivity
- The site list is *not* actually shared with DLA Site Notes in the browser — the apps use the same localStorage key but live on different domains, so the same `dla_sites.txt` must be imported into each app separately

## Future Plans / Ideas

*(No prior planning notes were on file when this status was written — items below are inferred from the codebase; correct or extend as needed.)*

- Possible cloud backup/sync of photos and entries (addresses the device-loss risk)
- Spreadsheet export (SheetJS is already loaded but unused)
- Continued visual alignment with the HGS Portal / DLA identity
