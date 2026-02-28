# HTML/JS to H5P Image Hotspots (v1.8) Conversion Guide

This guide is a repeatable process for converting your existing HTML/JS hotspot activities into Moodle-compatible `.h5p` files using **H5P.ImageHotspots 1.8.x**.

## 1) Confirm Moodle library version first

In Moodle (admin):

1. Go to **Site administration → H5P → Manage H5P content types**.
2. Confirm **Image Hotspots** is installed, and note the version (e.g. `1.8.3`).

> Important: Match your package dependency to the installed major/minor version (`1.8`), not a newer one like `1.10`, or import may fail.

---

## 2) Extract source data from HTML activity

From each HTML file, identify:

- `imageUrl` (e.g. `img/apple.jpg`)
- `imageAlt`
- `hotspots[]` with:
  - `x` (0–100)
  - `y` (0–100)
  - `title`
  - `message`

If your HTML uses a `PAGE_CONFIG_JSON` block, this maps directly and is ideal for conversion.

---

## 3) Build required H5P folder structure

For each activity, create:

```text
<activity-slug>/
  h5p.json
  content/
    content.json
    images/
      <background-image>.jpg
```

Example:

```text
apple-hotspots/
  h5p.json
  content/
    content.json
    images/
      apple.jpg
```

---

## 4) Create `h5p.json` (v1.8-compatible)

Use this template and update title only:

```json
{
  "title": "Apple Homepage Visual Analysis",
  "language": "en",
  "mainLibrary": "H5P.ImageHotspots",
  "embedTypes": ["div"],
  "license": "U",
  "defaultLanguage": "en",
  "preloadedDependencies": [
    { "machineName": "H5P.ImageHotspots", "majorVersion": 1, "minorVersion": 8 },
    { "machineName": "H5P.Text", "majorVersion": 1, "minorVersion": 1 },
    { "machineName": "FontAwesome", "majorVersion": 4, "minorVersion": 5 },
    { "machineName": "H5P.Transition", "majorVersion": 1, "minorVersion": 0 }
  ]
}
```

Notes:

- Keep `H5P.ImageHotspots` at `1.8` for Moodle installations using `1.8.x`.
- `H5P.Text` content is used for popup body text.

---

## 5) Create `content/content.json`

Minimum structure:

```json
{
  "iconType": "icon",
  "icon": "plus",
  "color": "#2563eb",
  "hotspots": [],
  "hotspotNumberLabel": "Hotspot #num",
  "closeButtonLabel": "Close",
  "containsAudioVideoLabel": "Contains Audio/Video",
  "backgroundImageAltText": "Apple Website Screenshot",
  "image": {
    "path": "images/apple.jpg",
    "mime": "image/jpeg",
    "copyright": {
      "license": "U"
    }
  }
}
```

For each source hotspot, add an item like:

```json
{
  "position": { "x": 50, "y": 24 },
  "alwaysFullscreen": false,
  "header": "Dominance (Principle)",
  "content": [
    {
      "params": { "text": "<p>The word 'iPhone' acts as the dominant element...</p>" },
      "library": "H5P.Text 1.1",
      "subContentId": "GENERATE-UUID-HERE",
      "metadata": {
        "contentType": "Text",
        "license": "U",
        "title": "Untitled Text"
      }
    }
  ]
}
```

Rules:

- Keep `x`/`y` in percentage space (0–100).
- Wrap body text in simple HTML (`<p>...</p>`).
- Use unique `subContentId` UUID values.

---

## 6) Package as `.h5p`

Zip the **contents of the activity folder** (not the parent directory), then rename `.zip` to `.h5p`.

The archive root must contain:

- `h5p.json`
- `content/content.json`
- `content/images/...`

If the archive contains an extra top-level folder, Moodle may reject it.

---

## 7) Validate before upload

Pre-upload checks:

1. `h5p.json` is valid JSON.
2. `content/content.json` is valid JSON.
3. `mainLibrary` is `H5P.ImageHotspots`.
4. Dependency version is `1.8` (for Moodle with 1.8.3 installed).
5. `content/image.path` points to an existing file in `content/images/`.
6. Every hotspot has `position`, `header` (optional but recommended), and one `H5P.Text 1.1` content item.

---

## 8) Upload to Moodle

1. In a course, add an **H5P activity** (or upload to Content bank).
2. Upload the `.h5p` file.
3. If Moodle reports missing libs, re-check dependency versions in `h5p.json`.

---

## 9) Common failures and fixes

### Error: Missing library / unsupported version

- Cause: Package targets wrong version (e.g. 1.10 while Moodle has 1.8.3).
- Fix: Set `H5P.ImageHotspots` to `majorVersion: 1`, `minorVersion: 8`, then rebuild.

### Error: Invalid package structure

- Cause: Zipped parent directory instead of package contents.
- Fix: Recreate archive so `h5p.json` is at archive root.

### Error: Background image not shown

- Cause: Wrong `image.path` or missing file.
- Fix: Ensure image exists at `content/images/<file>` and path matches exactly.

---

## 10) Fast conversion checklist (copy/paste)

1. Confirm Moodle Image Hotspots version.
2. Extract `imageUrl`, `imageAlt`, and hotspots from source JSON.
3. Build folder with `h5p.json` + `content/content.json` + image.
4. Set dependency `H5P.ImageHotspots` to `1.8`.
5. Convert each message to `H5P.Text 1.1` popup content.
6. Validate JSON and file paths.
7. Zip package correctly and rename to `.h5p`.
8. Upload and test each hotspot.

