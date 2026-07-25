[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on002181-blue)](https://doi.org/10.82901/nemar.on002181)

These are the EEG baseline data used in the study on the association between stunting and EEG brain functional connectivity in Bangladeshi children (https://doi.org/10.1101/447722).

Data with an ID <  2000 were collected for a cohort of  36-month-old toddlers, and those with an ID > 2000 were collected for a cohort of 6-month-old infants.  The children were watching screen savers for 2 minutes.

## NEMAR curation changes (2026-05-21, revised 2026-05-27)

The BIDS validator went from 226 errors + 4975 warnings to 0 errors + 3167 warnings. None of the raw `.set` files were modified — every change is to a text sidecar.

**Channel tables (`channels.tsv`, all 226 recordings)**
- The 124 scalp electrodes (E1–E124) had their channel type written in lowercase (`eeg`); BIDS expects the uppercase `EEG`, so they were capitalized.
- The `Cz` row was marked `type=unknown, units=unknown`. Cz is the recording reference (the `_eeg.json` files list `EEGReference: "Center(Cz)"`), so it now reads `type=MISC, units=n/a` — `unknown` is not an allowed channel type, and a reference channel carries no measured value.

**Recording sidecars (`_eeg.json`, all 226 recordings)**
- The channel-count field was spelled `MiscChannelCount`; BIDS uses all-uppercase `MISCChannelCount`. Renamed it (the value, `1`, was already correct) so the validator recognizes it.

**Two shared sidecars added at the dataset root** (so the same information isn't repeated in all 226 files; BIDS applies a root sidecar to every matching recording)
- `task-Baseline_eeg.json` — records the electrode layout (`EEGPlacementScheme: "EGI HydroCel GSN 128"`, which matches the 124 `E1–E124` electrodes plus `Cz` that every recording lists) and that the data is one `continuous` segment (each `.set` reports a single trial; the recordings are 2-minute screen-saver sessions).
- `task-Baseline_events.json` — describes the three event-table columns that were previously undocumented: `sample` (a sample index), `type` (always `trigger`), and `value` (either `onset` or `target`).

**Dataset description (`dataset_description.json`)**
- `BIDSVersion` was `1.2`, too old for the current validator; set to `1.11.1` (the version the validator checks against).
- Added `DatasetType: "raw"` so the dataset is validated as raw data rather than a derivative.
- Removed a placeholder source reference, `SourceDatasets: [{"DOI": "mockDOI"}]` — `mockDOI` is not a real DOI and would mislead anyone who followed it. The dataset's own DOI and the study link in this README are unchanged.

**Acquisition times (`scans.tsv`) — left exactly as published**
- EEGLAB/MNE adds a `.000000` microsecond suffix to the times when it reads the files, but the published values (e.g. `2019-09-15T14:38:13`) are already valid BIDS (fractional seconds are optional), so they were left unchanged.

**Important — the recording data is missing from this dataset**
- Every `.set` header points to a separate `.fdt` data file (e.g. `1473.s.1hzHighpass.fdt`) that is not present here, on NEMAR, or in the OpenNeuro source. The actual EEG signal lives in those `.fdt` files, so as published the recordings cannot be opened by MNE, EEGDash, or eegprep. The BIDS validator does not check for this, so it is not reported as an error, but the dataset is effectively unusable until the `.fdt` files are recovered from the original authors — a data-recovery step beyond what this metadata cleanup can do.

**Remaining warnings (3167) — left on purpose**
- These are all "recommended but missing" fields that need information from the study, lab, or equipment that isn't in the dataset (for example: manufacturer, cap model, head circumference, hardware filters, ground electrode, cognitive-atlas IDs). They were left blank rather than filled with guesses.
