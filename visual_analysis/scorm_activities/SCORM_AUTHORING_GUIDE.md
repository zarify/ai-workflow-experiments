# HTML/JS Hotspot to SCORM Visual Analysis — Authoring Guide

This guide explains how to create a new SCORM 1.2 activity based on an existing HTML/JS hotspot page. The result is a self-contained `.zip` file that can be uploaded to Moodle as a SCORM activity and report scores to the gradebook.

---

## How it works

The SCORM package is a folder of three things: a screenshot image, a manifest file, and a single `index.html`. The `index.html` contains two clearly separated sections:

1. **`PAGE_CONFIG_JSON`** — the activity data (title, image path, questions). This is the only thing you edit per activity.
2. **The engine** — the generic code that renders the UI, handles interactions, and communicates with Moodle. You never touch this.

When uploaded to Moodle:

- Students see the screenshot with numbered clickable markers.
- Clicking a marker opens a multiple-choice question panel.
- Answered markers turn **green** (correct) or **red** (incorrect).
- Once all markers are answered, the score is sent to the Moodle gradebook as a percentage (0–100).
- Moodle determines pass/fail by comparing the score against the `adlcp:masteryscore` value in the manifest.

---

## File structure

Every activity has this layout:

```
<activity-slug>/
  imsmanifest.xml       ← 3 lines to change per activity
  index.html            ← PAGE_CONFIG_JSON block at the top; engine below
  img/
    <screenshot>.jpg    ← replace with your image
```

Packaged as:

```
<activity-slug>.zip     ← upload this to Moodle
```

---

## Step-by-step: creating a new activity

### Step 1 — Copy the template

Duplicate the `apple-visual-analysis/` folder and rename it, e.g. `instagram-visual-analysis/`.

### Step 2 — Replace the image

Drop your screenshot into the `img/` folder. Remove the old one. The image can be any reasonable size; it scales to fit the browser window.

### Step 3 — Edit `PAGE_CONFIG_JSON` in `index.html`

Open `index.html`. At the very top of the `<script>` block you will see:

```json
{
  "title": "Apple Homepage — Visual Analysis",
  "imageUrl": "img/apple.jpg",
  "imageAlt": "Apple homepage screenshot",
  "hotspots": [ ... ]
}
```

Update the four top-level fields, then replace the `hotspots` array. Every hotspot is one question:

```json
{
  "x": 44,
  "y": 38,
  "header": "Emphasis: Contrast (Principle)",
  "context": "Look at the blue 'Learn more' button against the white background.",
  "question": "Which principle draws the viewer's eye to this button?",
  "choices": [
    { "text": "Emphasis through Contrast — saturated blue against a neutral background", "correct": true },
    { "text": "Emphasis through Proportion — the button is made larger than its surroundings", "correct": false },
    { "text": "Repetition — the same colour is used throughout to unify the design", "correct": false },
    { "text": "Proximity — the button is placed close to related text", "correct": false }
  ]
}
```

**Field reference:**

| Field | Required | Description |
|---|---|---|
| `x` | ✅ | Horizontal position of the marker, as a percentage of image width (0 = left edge, 100 = right edge) |
| `y` | ✅ | Vertical position of the marker, as a percentage of image height (0 = top, 100 = bottom) |
| `header` | ✅ | Short label shown as the marker title in the question panel (e.g. `"Dominance (Principle)"`) |
| `context` | — | Italic hint shown below the header, directing the student's eye (e.g. `"Look at the large heading…"`) |
| `question` | ✅ | The question stem |
| `choices` | ✅ | Array of 3–5 answer options. Exactly one must have `"correct": true`. Order doesn't matter — choices are shuffled at runtime. |

> **Finding x/y values:** Open the screenshot in a browser, right-click → Inspect, hover over the element, and read the approximate position as a percentage. Alternatively, open the original `*_homepage.html` activity — the `PAGE_CONFIG_JSON` in that file uses the same coordinate system and is a good starting point.

> **Validation:** If the JSON is malformed (wrong syntax, missing required field, zero or multiple `"correct": true` entries), the activity will display a clear error message in-page rather than silently breaking.

### Step 4 — Edit `imsmanifest.xml`

There are three lines to change. Search for the comments:

```xml
<!-- Change 1: unique slug for this activity, no spaces -->
<manifest identifier="instagram-visual-analysis" ...>

<!-- Change 2: human-readable name shown in Moodle -->
<title>Instagram Homepage Visual Analysis</title>

<!-- Change 3: match your image filename -->
<file href="img/instagram.jpg"/>
```

The `adlcp:masteryscore` line sets the pass mark (as a percentage). The default is 70 (i.e. 70% correct). Update it if needed:

```xml
<adlcp:masteryscore>70</adlcp:masteryscore>
```

> This is the **only** place the pass mark lives. Do not add a `masteryScore` field to `PAGE_CONFIG_JSON` — it is intentionally not there.

### Step 5 — Package as a zip

From inside the activity folder, run:

```bash
zip -qrD ../instagram-visual-analysis.zip .
```

The `-D` flag prevents directory entries being added to the archive, which Moodle requires. The dot (`.`) packages the **contents** of the folder, not the folder itself — `imsmanifest.xml` must be at the root of the archive.

Verify the contents look right:

```bash
unzip -l ../instagram-visual-analysis.zip
```

Expected output — three files, no directory entries:

```
  Archive:  instagram-visual-analysis.zip
    Length      Date    Time    Name
  ---------  ---------- -----   ----
      29916  ...               index.html
      99396  ...               img/instagram.jpg
       1753  ...               imsmanifest.xml
  ---------                   -------
                               3 files
```

---

## Uploading to Moodle

1. In your course, **Add activity → SCORM package**.
2. Upload the `.zip` file.
3. Under **Grading**, set:
   - **Grading method** → `Highest grade`
   - **Maximum grade** → `100`
4. Under **Completion tracking** (optional):
   - Enable **Require passing grade** to tie activity completion to the pass mark.
5. Save and display. Test by completing the activity as a student.

---

## How scoring reaches the gradebook

```
Student answers all questions
        ↓
Activity sends:   cmi.core.score.raw = 71  (percentage of correct answers)
                  cmi.core.lesson_status = "completed"
        ↓
Moodle compares:  score.raw ≥ adlcp:masteryscore (from manifest)
                  → "passed" or "failed"
        ↓
Gradebook shows:  71 / 100
```

With **Grading method: Highest grade** and **Maximum grade: 100**, the gradebook score equals the percentage directly — no scaling needed.

---

## Troubleshooting

**Activity shows a configuration error in-page**
The `PAGE_CONFIG_JSON` has a syntax error or a missing required field. Open the browser console for the specific message. Common causes: trailing comma after the last item in an array, a hotspot with two `"correct": true` entries, or a missing `"question"` field.

**Markers appear in the wrong position**
Re-check your `x`/`y` values. `x: 0` is the left edge, `x: 100` is the right. If you measured pixel positions from a differently-sized image, you need to convert: `x = (pixel_x / image_width) × 100`.

**Score not appearing in Moodle gradebook**
Check the SCORM activity's **Grading method** setting. If it is set to `Learning objects`, Moodle counts the number of SCOs with status "completed/passed" rather than the score — for a single-SCO package this gives 0 or 1, not the percentage. Switch to `Highest grade`.

**Activity shows as "failed" even with a good score**
The `adlcp:masteryscore` in `imsmanifest.xml` is higher than the student's score. Adjust the value and re-upload the package.

**Moodle rejects the zip on upload**
- Check the zip was created with `zip -qrD` (capital D, no directory entries).
- Check `imsmanifest.xml` is at the archive root (not inside a subfolder).
