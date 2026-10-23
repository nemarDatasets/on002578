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

## NEMAR curation changes (2026-05-21, revised 2026-05-27)

The BIDS validator went from 7 errors + 109 warnings to 0 errors + 95 warnings. None of the raw `.set`/`.fdt` recording files were modified — every change is to a text sidecar.

**Participant table (`participants.tsv`)**
- The header listed `participant_id` twice (`participant_id, gender, participant_id, age`), and the duplicate column just repeated the subject ID. The duplicate was dropped, so the header now reads `participant_id, gender, age` and the two rows read `sub-001 F 30` and `sub-002 M 30`.

**Dataset description (`dataset_description.json`)**
- `BIDSVersion` was `"v1.2.1"`, too old for the current validator (and the leading `v` is not canonical); set to `"1.11.1"`, the version the validator checks against.
- Added `DatasetType: "raw"` so the dataset is validated as raw data rather than a derivative.
- A generation-tooling entry (`GeneratedBy`) was left absent, exactly as the source published it. The original OpenNeuro dataset declares no such tooling, and this cleanup only fixes validator issues — which is not dataset generation — so nothing was invented here.

**Electrode tables (`sub-NNN_electrodes.tsv`, both subjects)**
- These were named `sub-NNN_task-attention_electrodes.tsv`. Electrode positions are a property of the subject, not of a task, so the `task-attention` part of the name was removed (a plain file rename — the contents are unchanged).

**Coordinate-system files (`sub-NNN_coordsystem.json`, both subjects)**
- Renamed the same way, from `sub-NNN_task-attention_coordsystem.json` to `sub-NNN_coordsystem.json`, for the same reason. The contents are unchanged; the `"CTF"` coordinate system is valid once the file sits at the subject level.

**Event description (`task-attention_events.json`) — new shared sidecar at the dataset root**
- The two subjects previously each carried a byte-identical per-subject `_events.json`, and there was also an orphan `events.json` at the root. These were replaced with a single `task-attention_events.json` at the dataset root, which BIDS automatically applies to both subjects.
- The time columns (`onset`, `duration`, `response_time`) gave their unit as `"second"`; changed to the BIDS-standard symbol `"s"`.
- Added a definition for the `sample` column, which was previously undocumented.
- Removed an obsolete list of allowed values (`Levels`) on the `value` column. Its old keys (`x11`..`x55`) matched no cells in `events.tsv`, which is why the validator rejected the plain numbers actually present (e.g. `34` on sub-001, `35` on sub-002).
- Kept the HED annotation dictionary that had been sitting in the orphan root `events.json` (12 entries: the numeric trigger codes 31, 33–35, 41–45, plus the labels `Picture`, `Response`, and `boundary`). In the orphan it was attached to a column named `type` that does not exist, so the annotations were unreachable; they are now attached to the real `value` column. Their content is unchanged.

**Removed files**
- The orphan root `events.json` was removed; its usable HED annotations were salvaged into the new `task-attention_events.json` (see above).
- The two per-subject `sub-001/eeg/sub-001_task-attention_events.json` and `sub-002/eeg/sub-002_task-attention_events.json` files were removed; they were byte-identical to each other and are now covered by the single inheriting root sidecar.
