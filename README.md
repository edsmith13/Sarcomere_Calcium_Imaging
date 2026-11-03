Reproducible companion code for the sarcomere-integrity and Ca²⁺-transient supplemental figures.

## Contents

```
paper_pipelines/
├── sarcomere_pipeline.ipynb        Z-disc detection + Pasqualini SPD / γ metrics
├── calcium_pipeline.ipynb          Calcium transient analysis (ND2 → dF/F₀ → metrics)
├── dsp_abundance_pipeline.ipynb    GFP-DSP puncta detection + abundance metrics
├── figures/                        populated by Run All
├── results/                        populated by Run All
├── cache/                          calcium pipeline's ND2 result cache
└── README.md                       (this file)
```

## Which notebook does what

### `sarcomere_pipeline.ipynb`
Fully automated. Detects Z-discs in α-actinin-GFP fluorescence images and computes:
- **SPD** (sarcomeric packing density) — Pasqualini/Grosberg framework, robust to tissue crinkling
- **γ** (Z-disk fraction) — fraction of foreground detected as elongated Z-disks vs. punctate Z-bodies
- Plus OOP, weighted OOP, sarcomere length (FFT), Z-disc density.

**To use**: edit `CONFIGS` in cell 6 to point at your data → Run All. Figures land in `figures/`, per-image metrics in `results/pm_<dataset>.csv`.

### `dsp_abundance_pipeline.ipynb`
Quantifies GFP-tagged desmoplakin abundance in cardiac microbundles. Reuses
the calcium pipeline's rectangular-EHT tissue segmentation + ROI framework,
and adapts the LoG blob detector from `sarcomere_pipeline.ipynb` for
compact DSP puncta at cell–cell junctions (keep *low*-aspect blobs
rather than elongated Z-discs). Nine-step workflow mirrors the calcium
pipeline — configure → file discovery → tissue-ROI contact sheet →
optional ROI picker → batch detect → puncta-overlay QC → stats + figure
→ snapshot. Supports paired (baseline/post) and single-timepoint studies.
Metrics: DSP integrated intensity per µm², puncta density, puncta area
fraction, mean puncta intensity, punctate-vs-diffuse fraction.

**To use**: edit the `BASE`, `LINES`, `TIMEPOINTS`, `UM_PER_PX` block
→ run cells top to bottom. If DSP images are co-registered with calcium
(same FOV), you can share ROIs by pointing `MANUAL_ROIS_PATH` at
`~/calcium_manual_rois.json`.

### `calcium_pipeline.ipynb`
Ca²⁺ transient analysis for ND2 recordings of cardiac microbundles. Nine-step workflow:
1. **STEP 0** — Environment: imports and core `process_and_verify_roi()` function
2. **STEP 1** — Configure: edit `BASE`, `LINES`, `TIMEPOINT_SUBFOLDERS`, `PAIR_MAP`
3. **STEP 2** — Discover files and pair by tissue ID
4. **STEP 3** — Contact sheet: visual QC of auto-detected tissue ROIs
5. **STEP 4** — Optional manual ROI picker for any bad auto-detections
6. **STEP 5** — Batch analysis (cached, resumable, parallel)
7. **STEP 6** — Trace-QC picker: click bad traces to exclude
8. **STEP 7** — Stats + 5-panel composite figure (beat rate, upstroke, Δ CaTD50, Δ CaTD90, Δ upstroke)
9. **STEP 8** — Snapshot everything for archival

**To use**: edit cell 6 (STEP 1) to point at your data → Run cells top to bottom. The contact-sheet (STEP 3) and manual-ROI picker (STEP 4) are optional but recommended for new datasets. Metrics + figure saved to home dir with a timestamp; snapshot cell (STEP 8) copies everything to `~/calcium_snapshots/<date>_<name>/`.

## Requirements

Python ≥ 3.10, with:
- `nd2` (ND2 reader)
- `numpy`, `scipy`, `pandas`, `matplotlib`
- `opencv-python`, `scikit-image`
- `ipywidgets` (only for the calcium pipeline's interactive QC pickers)

Both notebooks have a `%pip install` line at the top that installs everything.

## Data layout expected

### Sarcomere data
Either
- one folder per timepoint: `<BASE>/<timepoint>/<tissue>.nd2`, or
- one flat folder: `<BASE>/<tissue>.nd2` with timepoint patterns in the filename.

See the `CONFIGS` dict at the top of the notebook for both examples.

### Calcium data
`<BASE>/<LINE>/<TIMEPOINT_SUBFOLDER>/*.nd2` — one subfolder per cell line, one per timepoint per line. Filenames may optionally contain an experiment tag matched via the `PAIR_MAP` list.

## Verifying the pipelines work

Sarcomere pipeline (fully automated end-to-end):
```
cd paper_pipelines
jupyter nbconvert --to notebook --execute sarcomere_pipeline.ipynb --output sarcomere_pipeline.executed.ipynb --ExecutePreprocessor.timeout=600
```
Produces `figures/{et1,mava}.{png,svg,tiff}` and `results/pm_{et1,mava}.csv`.

Calcium pipeline is interactive (STEP 6 uses ipywidgets for QC), so it's not usually headless-executable end-to-end. Open it in Jupyter and Run All up through STEP 5, then use the widgets in STEP 6, then run STEPs 7–8.

## Citations

The sarcomere metrics implement the framework from:
- Pasqualini, F. S. *et al.* Structural phenotyping of stem cell-derived cardiomyocytes. *Stem Cell Reports* **4**, 340–347 (2015).
- Sutcliffe, M. D. *et al.* High-content analysis of sarcomeric structure in hiPSC-CMs. *Nat. Commun.* **9**, 5296 (2018).
- Toepfer, C. N. *et al.* SarcTrack. *Circulation Research* **124**, 1172–1183 (2019).

Z-disc detector (LoG + adaptive threshold + PCA per blob) is a custom implementation for α-actinin-GFP EHTs at ~0.16 μm/pixel.

Ca²⁺ pipeline uses `nd2`, `scipy.signal.find_peaks`, and `scipy.optimize.curve_fit` (exponential decay τ) — no ML models.
# Sarcomere_Calcium_Imaging
