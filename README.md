# MRC Nucleus Reslice

A Jupyter notebook for batch reslicing cryo-electron microscopy (cryo-EM) image volumes stored in MRC format. For each user-specified point on the nuclear surface, the program computes the local surface normal vector from interactively drawn tangent lines, then produces three resliced image stacks oriented along the normal and two orthogonal tangent directions.

---

## Features

- **Memory-efficient MRC access** — uses memory-mapping and crops a minimal subvolume into RAM; the full volume is never loaded.
- **Interactive tangent picking** — full-size pop-up windows with zoom/pan toolbar for precise placement of tangent lines on the nuclear envelope.
- **Automatic normal computation** — SVD-based best-fit normal from three orthogonal 2-D tangent vectors; condition number reported as a quality indicator.
- **Three resliced outputs per point** — along the surface normal (n̂), and along both in-plane tangent vectors (û, v̂).
- **Batch processing** — list of surface points supplied via an Excel spreadsheet; progress written back to the spreadsheet automatically.
- **Live progress bar** — elapsed time and ETA displayed during reslicing.
- **Text report** — normal vector components, polar angles, tangent vectors, and SVD values saved as a plain-text file for each point.

---

## Repository contents

```
mrc_normal_reslice_V8.ipynb       Main notebook (batch workflow)
surface_points_template.xlsx      Template spreadsheet for batch input
MRC_Reslice_User_Guide.docx       End-user instruction sheet
requirements.txt                  Python dependencies
README.md                         This file
```

---

## Requirements

### Python packages

```
pip install -r requirements.txt
```

### GUI toolkit (for pop-up windows)

One of the following must be installed:

| Toolkit | Install |
|---------|---------|
| Tk (usually bundled with Python) | — |
| Qt5 | `pip install PyQt5` |
| Qt6 | `pip install PyQt6` |

> **Note:** `ipympl` is **not** required. The notebook uses a native desktop backend (`TkAgg`, `Qt5Agg`, etc.) to open pop-up windows, which gives reliable mouse interaction without any Jupyter extension configuration.

---

## Quick start

1. **Clone the repository**

   ```bash
   git clone https://github.com/YOUR_USERNAME/mrc_nucleus_reslice.git
   cd mrc_nucleus_reslice
   ```

2. **Install dependencies**

   ```bash
   pip install -r requirements.txt
   ```

3. **Fill in the spreadsheet**

   Open `surface_points_template.xlsx` and enter the voxel coordinates (Z, Y, X) of each point on the nuclear surface you want to process. See the *Instructions* sheet inside the spreadsheet for column definitions.

4. **Open the notebook**

   ```bash
   jupyter lab mrc_normal_reslice_V8.ipynb
   ```

5. **Edit Cell 2** — set `MRC_PATH` to your MRC file and `POINTS_XLSX` to your filled-in spreadsheet.

6. **Run all cells in order** (Cells 1–5 are setup; Cell 6 runs the interactive batch loop).

---

## Workflow

```
Cell 1  →  Set GUI backend & import libraries
Cell 2  →  Configuration  (edit this cell)
Cell 3  →  Inspect MRC header
Cell 4  →  Load & validate surface-points spreadsheet
Cell 5  →  Define helper functions
Cell 6  →  Batch loop (interactive tangent picking + reslicing)
```

For each point in the spreadsheet, Cell 6:

1. Extracts XY, XZ, and YZ orthogonal slices and shows them in a reference pop-up.
2. Opens three sequential tangent-picker pop-ups (XY → XZ → YZ). In each window, left-click two points along the nuclear envelope to define the tangent line. Right-click to redo. Use the toolbar zoom/pan tools to navigate, then deactivate them before clicking tangent points.
3. Computes the surface normal via SVD and saves a text report.
4. Reslices the volume along n̂, û, and v̂ using subvolume cropping and trilinear interpolation.
5. Saves three MRC files and marks the spreadsheet row as `done`.

Points already marked `done` in the spreadsheet are automatically skipped when Cell 6 is re-run.

---

## Output files

For each point with identifier `XX`:

| File | Description |
|------|-------------|
| `resliced_normal_XX.mrc` | Stack resliced along the surface normal (n̂) |
| `resliced_tangent_u_XX.mrc` | Stack resliced along the first tangent vector (û) |
| `resliced_tangent_v_XX.mrc` | Stack resliced along the second tangent vector (v̂) |
| `normal_vector_XX.txt` | Text report: point coordinates, tangent vectors, SVD values, normal vector, polar angles |
| `ortho_slices/point_XX/` | XY, XZ, YZ reference TIFFs (if `SAVE_ORTHO_TIFFS = True`) |

All MRC files embed the full geometry (origin, n̂, û, v̂) as label strings in the file header.

---

## MRC coordinate convention

The notebook follows the standard MRC axis convention: data is stored as `(Z, Y, X)` with Z as the slowest (outermost) axis. All coordinates in the spreadsheet and in the output file labels use this `(Z, Y, X)` order.

---

## Acknowledgements

Built with [mrcfile](https://github.com/ccpem/mrcfile), [NumPy](https://numpy.org), [SciPy](https://scipy.org), [Matplotlib](https://matplotlib.org), [pandas](https://pandas.pydata.org), and [openpyxl](https://openpyxl.readthedocs.io).
