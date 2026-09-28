# Audit Photo Collector — Status

**Last updated:** 2026-09-28

## Status: ✅ Fully Functional (Deployed & In Use)

The app is stable, deployed to Azure Static Web Apps, and suitable for field use. The latest work (2026-08-04/05) hardened photo safety and export tracking: capture now verifies each photo actually saved (red "NOT SAVED" badge + toast on failure), a header integrity badge reports photo-storage health, both exports warn before shipping entries with missing photos, and entries included in a completed ZIP export are stamped and badged "exported" in the saved list. A 2026-09-28 code review hardened this further: filtered exports no longer delete other entries' only-copy fallback photos, entries missing a photo are never stamped exported, the missing-photo check runs before the export is built, Save & New waits for pending photo writes and asks before filing a failed one, export stamps survive text edits and backup restores, and batch delete shows export status. Current version is v2.10 (2026-07-31): tapping a filled photo slot now opens the photo full size for field verification (Retake moved into the viewer; empty slots still capture on tap). v2.9 added compass facing direction captured per photo (magnetometer sampled when the photo slot is tapped, corrected to true north via a CONUS declination table, 8-point cardinal labels, low-confidence readings suppressed) — shown on slot badges, the photo viewer, Photo Log captions, and a Photo Facing column in CSV/XLSX. v2.8 moved the satellite Site Map to the DLA Site Notes app (see dla-site-notes/SITE-MAP-INSTRUCTIONS.md). v2.4 added readable thumbnails; v2.3 added site/date filtering, site-tagged exports, export log, and batch operations.

## Summary

Audit Photo Collector is a mobile-first Progressive Web App built for HGS Engineering field staff to collect geotagged photos during environmental site audits. It runs entirely in the browser as a single-page app (no build step, no backend) and works offline after the first load thanks to a service worker. In the field, an auditor captures photos per location entry (two default slots plus unlimited extras), and the app automatically records GPS coordinates, timestamps, location/details text, and resizes photos client-side at a configurable resolution and quality. A "Save & New" workflow lets the auditor move quickly between locations, with site-name autocomplete from an importable site list. Back at the office, the app exports a formatted Word-document Photo Log — one photo per page with sequential reference numbers (e.g., C2-001), location, details, GPS coordinates, and timestamp — ready for inclusion in audit reports. Entries are stored locally (localStorage for metadata, IndexedDB for full-resolution photos), with a JSON backup/import feature for entry metadata. It is deployed via GitHub Actions to Azure Static Web Apps; a companion app, DLA Site Notes, was split out into its own repository, and this app redirects its old `/notes.html` path there.

## Capabilities

- Photo capture with client-side resizing (720p–1440p presets, quality slider)
- Photo-safety checks (2026-08-04): capture verifies each photo was actually stored (red "NOT SAVED" badge + toast on failure); a header integrity badge shows "N photos safe" / "X of N missing" (tap to list affected entries); the app requests persistent storage; full-res photos are no longer duplicated into localStorage when IndexedDB works
- Readable previews: 240px stored thumbnails, form slots hydrate to the full-resolution photo, and both form slots and saved-list thumbnails are tap-to-view (full-size viewer with facing caption; form-slot viewer includes a Retake button); old low-res thumbnails regenerate automatically from stored photos on launch
- Automatic + manual GPS coordinate capture per entry (an external Bluetooth GPS receiver improves accuracy automatically via iOS Location Services)
- Compass facing direction per photo: magnetometer heading sampled at photo-slot tap (workflow: point the device, then tap), landscape-corrected, converted magnetic-to-true with an embedded CONUS declination grid; 8-point cardinal + degrees in Photo Log captions, viewer, and a Photo Facing export column; readings with compass-reported error over 50 deg are suppressed, 30-50 deg marked "~". Requires one iOS motion-permission tap; accuracy is approximate (typically 10-25 deg, worse near steel structures)
- Save & New workflow with site-name autocomplete (importable `.txt` site list); warns before saving an entry with no site name
- Edit previously saved entries; saved list shows each entry's site name (amber "No site" warning when missing)
- Site + date filters on the saved list and in the Export dialog (both ZIP and Photo Log exports respect them)
- Batch operations on the filtered set: **Set Site** (assign a site name to many entries at once) and **Delete** (removes entries and their stored photos)
- ZIP export: photo files named `P0001_Site_Name.jpg`; ZIP/report/CSV filenames carry the site name when a single site is exported
- Word (.docx) Photo Log export with sequential photo reference numbers; captions include a "Site:" line, and the banner/header/filename show the site when the export covers one site
- Export integrity guard (2026-08-04): both ZIP and Photo Log exports refuse to silently ship missing photos — a warning naming the affected entries blocks the export unless acknowledged
- Per-entry export tracking (2026-08-05): entries included in a completed ZIP export are stamped `exportedAt`; the saved list badges each entry green "exported" / amber "not exported", and both single and batch delete confirms show export status so nothing unexported gets deleted blind; entries with a photo missing from the ZIP are not stamped, and a retake or new photo after export clears the stamp (the Photo Log is a report and does not stamp)
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
- Native wrap via Capacitor: a filesystem storage layer shipped with the 2026-08-04 photo-safety work but is inert on web — wrapping the app would activate it for real photo durability
- Continued visual alignment with the HGS Portal / DLA identity
