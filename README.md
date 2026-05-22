[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on002181-blue)](https://doi.org/10.82901/nemar.on002181)

These are the EEG baseline data used in the study on the association between stunting and EEG brain functional connectivity in Bangladeshi children (https://doi.org/10.1101/447722).

Data with an ID <  2000 were collected for a cohort of  36-month-old toddlers, and those with an ID > 2000 were collected for a cohort of 6-month-old infants.  The children were watching screen savers for 2 minutes.  

## NEMAR curation changes (2026-05-21)

BIDS validator: 226 errors + 4975 warnings -> 0 errors + 3166 warnings. Raw `.set` binary payloads unchanged (all modifications are sidecars / text TSVs).

### `sub-*/sub-*_scans.tsv` (all 226 files)
- Rewrote `acq_time` from `YYYY-MM-DDTHH:MM:SS` to `YYYY-MM-DDTHH:MM:SS.000000`. Why: `mne_bids.read_raw_bids` (and other downstream BIDS readers) parses `acq_time` with `strptime('%Y-%m-%dT%H:%M:%S.%f')` and rejects timestamps without fractional seconds; appending `.000000` keeps the original second-precision timestamp byte-equivalent while satisfying the parser. No clock value changed.

### `sub-*/eeg/sub-*_task-Baseline_channels.tsv` (all 226 files)
- Changed `type` column for E1..E124 rows from lowercase `eeg` to canonical `EEG`. Why: closes the only validator error (`TSV_VALUE_INCORRECT_TYPE:type`, 226x) — BIDS-EEG schema (`rules.tabular_data.eeg.EEGChannels`) requires uppercase channel-type tokens.
- Changed `Cz` row from `type=unknown, units=unknown` to `type=MISC, units=n/a`. Why: Cz is the recording reference (see `EEGReference: "Center(Cz)"` in each `_eeg.json`); BIDS does not accept `unknown` as a channel type. `MISC` matches the existing `MISCChannelCount: 1` declaration. Reference channels carry no measured signal value, so `n/a` is the correct unit token.

### `sub-*/eeg/sub-*_task-Baseline_eeg.json` (all 226 files)
- Renamed key `MiscChannelCount` (mixed case) to canonical `MISCChannelCount`. Why: BIDS-EEG schema uses all-uppercase `MISC` in the key name; the misspelt key was invisible to the validator, which then warned that `MISCChannelCount` was missing (226x `SIDECAR_KEY_RECOMMENDED:MISCChannelCount`). The value `1` was already correct.

### `task-Baseline_events.json` (new, inheriting root sidecar)
- Created with `Description` entries for the three non-canonical event-TSV columns (`sample`, `type`, `value`). Why: closes 678x `TSV_ADDITIONAL_COLUMNS_UNDEFINED` warnings (3 columns x 226 recordings) in a single edit via BIDS inheritance. Column meanings derived from inspecting the event TSVs themselves: `type` is always `trigger`; `value` is exactly two distinct strings (`onset` once per recording, `target` for every stimulus event); `sample` is a monotonically-increasing integer index. A `Levels` enum is declared for `value` only — `type` and `sample` are free-form because the spec gives no benefit to enumerating a single-valued or sample-indexed column.

### `task-Baseline_eeg.json` (new, inheriting root sidecar)
- Added `EEGPlacementScheme: "EGI HydroCel GSN 128"`. Why: the per-recording `channels.tsv` files are byte-identical and list 124 electrodes named `E1..E124` plus a `Cz` reference — the unambiguous channel layout of the EGI HydroCel Geodesic Sensor Net 128. Closes 226x `SIDECAR_KEY_RECOMMENDED:EEGPlacementScheme`.
- Added `RecordingType: "continuous"`. Why: each `.set` header reports `EEG.trials=1` (verified during triage) and the README describes a single 2-minute screen-saver recording with no epoching, so all 226 recordings are single continuous segments. Closes 226x `SIDECAR_KEY_RECOMMENDED:RecordingType`.

### `dataset_description.json`
- Bumped `BIDSVersion` from `1.2` to `1.9.0`. Why: closes `UNKNOWN_BIDS_VERSION`; `1.2` predates the BIDS-EEG extension's stable revisions and is not recognized by the current validator schema.
- Added `DatasetType: "raw"`. Why: pairs with `GeneratedBy` to prevent the validator from cascading into derivative-rules and spuriously warning about missing-Description fields on every sidecar.
- Added `GeneratedBy: [{"Name": "nemar-cli", "Version": "0.8.8", "CodeURL": "https://github.com/nemar-org/nemar-cli"}]`. Why: closes `JSON_KEY_RECOMMENDED:GeneratedBy`; documents the curation/rehost provenance.
- Removed `SourceDatasets: [{"DOI": "mockDOI"}]`. Why: `mockDOI` is a non-resolvable placeholder string, not a real DOI. Leaving it would mislead any consumer following the field; removing it is honest. (The dataset's own DOI is preserved in `DatasetDOI`; the study publication DOI is preserved in the README link.)

### Remaining warnings (3166 total, Tier-C — left intentionally)

All remaining warnings are `SIDECAR_KEY_RECOMMENDED` entries that require external (study/lab/equipment) information not present in the dataset and not defensibly inferable: `Manufacturer`, `ManufacturersModelName`, `SoftwareVersions`, `DeviceSerialNumber`, `Instructions`, `CapManufacturer`, `CapManufacturersModelName`, `EEGGround`, `HardwareFilters`, `HeadCircumference` (per-subject), `SubjectArtefactDescription`, `CogAtlasID`, `CogPOID`, `StimulusPresentation` (226x each = 3164), plus `HEDVersion` (no HED tags in the dataset) and `SourceDatasets` (no upstream BIDS dataset to declare). Filling these with placeholders would invent metadata; they remain pending lab-side input.

### Out of mechanical scope (not addressed by this pass)

- The 226 `.set` files reference external `.fdt` payloads (`EEG.data` is the string `'<N>.s.1hzHighpass.fdt'`) that are missing from the NEMAR/OpenNeuro mirror. The validator does not inspect `.set` payload integrity and so does not flag this, but the dataset is currently unusable to downstream readers (EEGDash, eegprep, MNE) for that reason. Recovering or re-exporting the `.fdt` payloads is a binary-side rehost task that this metadata curation pass cannot address.
