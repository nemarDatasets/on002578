[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on002578-blue)](https://doi.org/10.82901/nemar.on002578)

Data for this selective attention task was collected in 2004
at the Swartz Center for Computational Neuroscience at UCSD.
These datasets are part of a larger corpus of 32-channel data
collected a few years prior. The experiment is identical
although the number of channel is larger (256), the electrode
positions are scanned and the anatomical MRI is provided
(allowing for precise source localization). See publication
for more details.

Raw data manipulation before export:
- Fuse all BDF BIOSEMI files and reference to electrode 135 (see loadallbdf_2020.m)
- Fuse with presentation file information (see loadallbdf_2020.m)
- Remove spurious events of type 'condition' and '201' (see clean_events.m)
- Add HED tags (see addHEDTags.m)
- Convert MRI to NIFTI format (MRIcron) and reorient (MRIcrogl) (see convert_nifti.m)

## NEMAR curation changes (2026-05-21)

BIDS validator: 7 errors + 109 warnings → 0 errors + 94 warnings. Raw `.set`/`.fdt` binary payloads unchanged.

### `participants.tsv`
- Header had `participant_id` listed twice (`participant_id, gender, participant_id, age`), and the duplicate column repeated the subject ID. Dropped the duplicate so the header is `participant_id, gender, age`. Two data rows now read `sub-001 F 30` and `sub-002 M 30`. Closes the `TSV_COLUMN_HEADER_DUPLICATE`, `TSV_COLUMN_MISSING`, and `PARTICIPANT_ID_MISMATCH` errors.

### `dataset_description.json`
- `BIDSVersion`: `"v1.2.1"` → `"1.8.0"` (BIDS-canonical, no leading `v`). Closes `UNKNOWN_BIDS_VERSION`.
- Added `DatasetType: "raw"`.
- Added `GeneratedBy: [{Name: "nemar-cli", Version: "0.8.8", CodeURL: "https://github.com/nemar-org/nemar-cli"}]`.

### `sub-NNN/eeg/sub-NNN_task-attention_electrodes.tsv` (2 files)
- `git mv` to `sub-NNN/eeg/sub-NNN_electrodes.tsv` (dropped the `task-` entity — electrode positions are subject-level, not task-level per BIDS). Closes `EXCESSIVE_ELECTRODE_SPECIFICITY` ×2 and dependent `SIDECAR_KEY_RECOMMENDED:TaskName` warnings.

### `sub-NNN/eeg/sub-NNN_task-attention_coordsystem.json` (2 files)
- `git mv` to `sub-NNN/eeg/sub-NNN_coordsystem.json` (same rationale). Closes `EXCESSIVE_COORDSYSTEM_SPECIFICITY` ×2 and `JSON_SCHEMA_VALIDATION_ERROR:EEGCoordinateSystem` (file contents unchanged; value `"CTF"` is a valid BIDS coordinate system once the file is at the subject-level path).

### `task-attention_events.json` (new, consolidating root sidecar)
- Created at dataset root, replacing the two byte-identical per-subject `_events.json` files (removed) plus the orphan root `events.json` (also removed). BIDS inheritance carries this single file to both subjects.
- All time columns (`onset`, `duration`, `response_time`): `"Units": "second"` → `"Units": "s"` (BIDS-canonical SI symbol).
- Added `sample` column definition (was undocumented).
- `value` column: removed the obsolete `Levels` block — the old keys `x11..x55` did not match any cells in `events.tsv`, which is what triggered the `TSV_VALUE_INCORRECT_TYPE` errors on bare numerics like `34` (sub-001 line 249) and `35` (sub-002 line 252). Closes both errors.
- `value` column: preserved the HED dictionary from the orphan root `events.json` (12 entries: numeric trigger codes 31/33-35/41-45 plus the string classes `Picture`, `Response`, `boundary`) and re-anchored it under `value.HED` to match the actual column name in `events.tsv`. The orphan had nested it under a non-existent column `type`, so the annotations were unreachable; their content is unchanged.

### Removed files
- `events.json` (root) — orphan that nested HED tags under a non-existent column `type`. Salvageable HED entries now live in the new `task-attention_events.json` under `value.HED`. Removing this file also closes the `HED_ERROR:SIDECAR_INVALID` on sub-001.
- `sub-001/eeg/sub-001_task-attention_events.json`, `sub-002/eeg/sub-002_task-attention_events.json` — byte-identical to each other; both replaced by the inheriting root sidecar.
