# Siemens FLC QC Viewer

Browser-based, single-file tool for **quality control of Siemens flat-panel / FLC images** and **uncompressed DICOM images**.

The application runs locally in the browser, requires no installation and has no external JavaScript dependencies. It provides image viewing, ROI statistics, batch/range analysis and dedicated workflows for detector quality-control measurements.

> Current UI version: **Siemens FLC QC Viewer v12 — DICOM + ROI Range**

---

## Main features

- Load Siemens FLC acquisitions as:
  - `.hdr` + raw image without extension
  - `.hdr` + `.pp`
  - combined/raw raster with automatic offset detection
- Load **uncompressed monochrome DICOM** (`.dcm` / `.dicom`)
- Automatic detection of:
  - image matrix
  - pixel spacing when available
  - DICOM Window Center / Window Width
  - Siemens FLC geometry when available in the header
- Manual **Window / Level**
- Automatic W/L
- Image inversion
- Zoom control
- Rectangular ROI
- Elliptical ROI
- Distance measurement
- ROI statistics:
  - number of pixels
  - area
  - mean
  - standard deviation
  - minimum
  - maximum
  - coefficient of variation
- ROI templates:
  - save in browser
  - export to JSON
  - import from JSON
  - automatic rescaling when the image matrix changes
- Batch/range analysis using the same ROI coordinates on multiple images
- Excel-friendly clipboard export
- Response-function and reproducibility tables
- Detector uniformity analysis
- Bad-pixel search
- Ghost / latent-image measurement workflow
- Integrated **“Guida Galattica”** help poster
- Fully local processing: the application code makes no network requests

---

## Quick start

No build step is required.

1. Download or clone the repository.
2. Open:

```text
SIEMENS_TLC_CalculatorHDR.html
```

in a modern browser.

A Chromium-based browser such as **Microsoft Edge** or **Google Chrome** is recommended, especially for folder selection and batch analysis.

The application can be opened directly using `file://`; no web server is required.

---

## Loading images

### Siemens FLC

For a single acquisition, load the corresponding files together:

```text
image.hdr
image
```

or:

```text
image.hdr
image.pp
```

The viewer attempts to read the FLC header, determine image geometry and recover the pixel spacing.

Supported built-in matrix presets include:

| Preset | Matrix |
|---|---:|
| MAX mini | 1920 × 1520 |
| MAX wi-D | 2350 × 2866 |
| MAX static | 2868 × 2874 |
| MAX dynamic RAD | 2840 × 2874 |
| MAX dynamic FLU Z0 | 880 × 960 |
| MAX dynamic FLU/DFR Z1/Z3 | 1024 × 1024 |
| MAX dynamic FLU/DFR Z2 | 768 × 768 |
| MAX dynamic DFR Z0 | 1320 × 1440 |
| FLC storage detected | 3072 × 2657 |

If the raster geometry does not match a preset, the application also tries to infer a compatible UInt16 matrix from the payload size.

The raster offset is automatically tested at:

```text
0 bytes
10240 bytes
```

and can also be edited manually.

### DICOM

The built-in DICOM parser currently supports:

- Implicit VR Little Endian
- Explicit VR Little Endian
- Explicit VR Big Endian
- 8-bit and 16-bit pixel data
- signed and unsigned pixels
- `MONOCHROME1`
- `MONOCHROME2`

The viewer reads, when available:

- Rows / Columns
- Pixel Spacing
- Window Center / Width
- modality
- acquisition date and time
- manufacturer and model
- station name
- kVp
- exposure time
- tube current
- exposure
- SID
- Study / Series / SOP identifiers

### DICOM limitations

The current version does **not** decode encapsulated/compressed pixel data.

The following transfer syntaxes therefore require an additional decoder and are explicitly rejected:

- JPEG
- JPEG-LS
- JPEG 2000
- RLE

ROI measurements use the **stored pixel values**. DICOM `Rescale Slope` and `Rescale Intercept` are displayed when present but are **not applied** to ROI statistics.

---

## ROI tools

Three measurement tools are available.

### Rectangular ROI

Returns:

```text
N pixels
Mean
SD
Min
Max
CV %
Area [mm²]   (when pixel spacing is known)
```

### Elliptical ROI

Uses the same statistics while including only pixels inside the ellipse.

### Distance

Returns:

```text
Δx [px]
Δy [px]
distance [px]
distance [mm]   (when pixel spacing is known)
```

### ROI editing

A selected ROI can be moved with the mouse or keyboard.

| Command | Action |
|---|---|
| Arrow keys | move 1 pixel |
| Shift + Arrow | move 10 pixels |
| Ctrl/Cmd + D | duplicate ROI |
| Delete / Backspace | delete ROI |

---

## ROI templates

ROI layouts can be reused between acquisitions.

Available operations:

- **Save in browser**
- **Recall**
- **Export JSON**
- **Import JSON**

The JSON template stores:

- matrix size
- pixel spacing
- ROI type and coordinates
- normalized ROI coordinates

Normalized coordinates allow a template to be adapted when the new image has a different matrix size.

Example structure:

```json
{
  "format": "FLC-QC-ROI",
  "version": 1,
  "matrix": {
    "w": 2840,
    "h": 2874
  },
  "pixelSpacing": {
    "x": "0.148",
    "y": "0.148"
  },
  "rois": []
}
```

> Some browsers may restrict `localStorage` when an HTML file is opened through `file://`. In that case, use **Export JSON / Import JSON**.

---

## Batch and range analysis

Use **Carica cartella / range** to load a sequence of acquisitions.

For Siemens FLC data, choose the expected pairing mode:

```text
.hdr + file without extension
```

or:

```text
.hdr + .pp
```

A folder containing uncompressed DICOM files can also be loaded as a series.

After loading:

1. choose the first and last image of the range;
2. press **Applica range**;
3. draw one or more ROIs;
4. press **Analizza range**.

The same ROI coordinates are measured on every image in the selected range.

For each ROI the table reports:

```text
image number
area
mean
standard deviation
minimum
maximum
```

The data can be copied directly as tab-separated values for Excel.

### QC shortcuts

The current workflow provides dedicated copy commands for:

- **Response function:** images 1–6
- **Reproducibility:** images 7–8

These image numbers reflect the current QC workflow and can be changed in the source code if a different acquisition protocol is used.

---

## Response function and reproducibility

A selected ROI can be added manually to the QC tables together with the entered dose.

Stored values are:

| Field |
|---|
| Dose [µGy] |
| Image number |
| Area |
| Mean |
| Standard deviation |
| Minimum |
| Maximum |

---

## Detector uniformity

The uniformity workflow is designed around the current AIFM-style acquisition procedure implemented in the application.

Default parameters:

```text
ROI side: 30 mm
ROI step: 15 mm
Image margin: 60 mm
```

The margin and ROI size can be edited from the interface.

The guided workflow asks for two images, defaulting to:

```text
2.5 mGy  → image 2
10 mGy   → image 4
```

For each dose the application calculates:

- `NULS`
- `NUGS`
- `NULSNR`
- `NUGSNR`

### Global non-uniformity

For a matrix of local ROI values:

```text
NUGS = (max - min) / ((max + min) / 2)
```

### Local non-uniformity

The local metric is the maximum relative difference between horizontally or vertically adjacent ROIs:

```text
NULS = max( |v - v_neighbour| / v )
```

The same calculation is applied to the ROI SNR matrix to obtain `NULSNR` and `NUGSNR`.

The resulting values can be copied directly to Excel.

---

## Bad-pixel search

The bad-pixel analysis is run on the first image selected for the uniformity test.

The current implementation reproduces the low-threshold logic used by the reference ImageJ macro:

```text
threshold = mean - 7 × SD
```

Pixels satisfying:

```text
pixel value <= threshold
```

inside the internal detector ROI are counted as bad pixels.

The interface reports:

- ROI size
- mean
- SD
- threshold
- minimum
- maximum
- number of detected bad pixels
- coordinates and values of detected pixels

Only the **low-value threshold** is currently implemented.

---

## Ghost / latent-image workflow

The Ghost tool uses two measurement ROIs and two user-selected images:

1. irradiated / high-contrast image
2. dark / minimum-load image

Default image numbers are:

```text
irradiated: 9
dark:       10
```

but both can be changed before the analysis.

The first two measurement ROIs are reused at the same coordinates on both images, producing four measurement rows:

```text
ROI1immIRR
ROI2immBuio
ROI3immIRR
ROI4immBuio
```

For each row the tool extracts:

```text
image number
area
mean
standard deviation
minimum
maximum
```

The four rows can be copied directly to Excel.

> The current implementation extracts the four ROI measurements needed by the workflow; it does not calculate an additional final ghost index inside the browser.

---

## Siemens FLC header handling

For compatible FLC headers, the viewer attempts to extract acquisition and image information including:

- date
- time
- protocol
- anatomy code
- processing fields
- detector / RF identifiers
- serial information
- UID strings
- matrix dimensions
- pixel pitch

The FLC geometry block is checked for internal consistency before it is accepted.

Some semantic header fields are intentionally marked in the interface as **still requiring confirmation across additional acquisitions**.

---

## Privacy and data handling

The application is self-contained.

- image files are read with browser file APIs;
- calculations run in JavaScript in the local browser session;
- the application source contains no `fetch`, XHR or WebSocket network calls;
- no image is intentionally uploaded by the application;
- the embedded guide is stored directly inside the HTML file.

Clipboard export and optional browser `localStorage` are the only browser-side persistence/convenience mechanisms used by the current version.

This makes the tool suitable for workflows in which QC images should remain on the local workstation, subject to the security policies of the institution and browser in use.

---

## Known limitations

- Compressed DICOM is not supported.
- DICOM color images are not supported.
- DICOM ROI statistics use stored pixel values; rescale slope/intercept are not applied.
- Folder DICOM series are ordered by their filename/path rather than by a complete DICOM series-management layer.
- The current UI does not provide a dedicated multi-frame DICOM frame navigator.
- Siemens proprietary header parsing is based on the currently identified FLC layout and should be validated on additional system/software versions.
- QC results should be checked against the acquisition protocol and institutional reference implementation before being used operationally.
- Browser permissions may affect clipboard access and `localStorage` when running from `file://`.

---

## Suggested QC workflow

```text
Load images
    ↓
Verify matrix and pixel spacing
    ↓
Check Window / Level
    ↓
Draw or import ROI template
    ↓
Analyze range
    ↓
Copy response / reproducibility data
    ↓
Run uniformity + bad-pixel analysis
    ↓
Run Ghost measurements
    ↓
Paste results into the QC spreadsheet
```

The integrated **🚀 GUIDA GALATTICA** button provides an in-application visual overview of the workflow.

---

## Project structure

A minimal repository can simply contain:

```text
.
├── SIEMENS_TLC_CalculatorHDR.html
└── README.md
```

No package manager, compiler or external assets are required.

---

## Development notes

The application is intentionally distributed as a **single HTML file** containing:

- HTML interface
- CSS
- JavaScript
- DICOM parser
- Siemens FLC parser
- ROI/QC calculation logic
- embedded help image

This keeps deployment simple on QC workstations and makes the tool usable without internet access.

If the project grows, possible future refactoring could separate:

```text
src/
  dicom.js
  flc.js
  roi.js
  qc.js
  ui.js
```

while retaining a compiled standalone HTML release for clinical-physics workstations.

---

## Validation

This software is a technical QC utility and should be validated locally against:

- known reference images;
- the existing spreadsheet or ImageJ workflow;
- expected ROI statistics;
- detector-specific acceptance criteria;
- the institution's quality-control procedure.

Automated regression tests using anonymized reference images would be a useful next step.

---

## Contributing

Issues and pull requests are welcome for:

- additional Siemens FLC variants;
- DICOM compatibility improvements;
- QC calculation validation;
- UI improvements;
- export formats;
- automated tests.

When reporting a parsing problem, include where possible:

```text
browser/version
file type
image matrix
transfer syntax (for DICOM)
expected result
observed result
```

Avoid uploading identifiable patient images to public issues.

---

## Disclaimer

This project is intended as a **quality-control and technical analysis tool**. It is not a diagnostic DICOM viewer and is not a substitute for validated institutional QC procedures, manufacturer software or applicable regulations.

