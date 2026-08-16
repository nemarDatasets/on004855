[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on004855-blue)](https://doi.org/10.82901/nemar.on004855)

FT dataset

This is a placeholder dataset.

## NEMAR curation changes (2026-05-21, revised 2026-05-27)

The BIDS validator went from 1 error + 47 warnings to 0 errors + 37 warnings. None of the raw `.set` or `.fdt` data files were modified; every change is to a text sidecar.

The single pre-curation error was a HED duplicate-tag error in the event annotations, and it is shared across the entire STRONG cohort (on004849/50/52/53/54/55 all carry the byte-identical defect). The mechanism: `sub-001_task-nback_events.tsv` has 197 onsets where two rows share the same `onset` time (BIDS allows duplicate onsets). When two rows share an `onset`, the HED validator merges their HED annotations into one combined string per time bucket. The HED dictionary in `task-nback_events.json` assigns the tag `"Task"` to about 80 of the roughly 107 distinct event codes, so whenever two rows with different codes that both map to `"Task"` share an `onset`, the merged string becomes `"Task,Task"` and the validator legitimately fires a duplicate-tag error. Of the 197 collisions, 196 are between code pairs whose HED tags collide on `"Task"` (the (1, 1103), (1, 1113), and (2096, 307) pairs); only the (3, 55011) pair has two distinct HED tags and merges cleanly.

**`dataset_description.json`**
- Added `DatasetType: "raw"` so the dataset is validated as raw data rather than a derivative.
- Updated `BIDSVersion` from `1.8.0` to `1.11.1` (the version the current validator checks against).
- `ReferencesAndLinks` was `[""]` (a list with one empty string), which is not a valid reference; it is now an empty list `[]`.
- `GeneratedBy` was left absent, exactly as the source published it. Nothing was added there.

**`sub-001/eeg/sub-001_task-nback_channels.tsv`**
- All 64 rows had `type=n/a` and `units=n/a`. These are scalp EEG channels named with 10-10 labels, so the type was set to `EEG` and the units to `uV` (microvolts, the standard unit for EEG). Channel names were not touched.

**`sub-001/eeg/sub-001_task-nback_eeg.json`**
- Added `MISCChannelCount: 0` and `TriggerChannelCount: 0` to make the channel-count bookkeeping explicit (no miscellaneous or trigger channels are present).
- Added `EEGPlacementScheme: "10-10"` because the channel names in `channels.tsv` are standard 10-10 system labels.
- All other keys were left unchanged.

**`task-nback_events.json`**
- Added a top-level `sample` column definition, because `events.tsv` has a `sample` column that was previously undocumented.
- Dropped the `HED` entries for event codes `"1"` and `"307"` from the `value` column (105 of the original 107 HED entries are preserved). Those two codes appear in `events.tsv` only at onsets where they collide with another code that is already tagged `"Task"`; removing just their HED entries breaks the `"Task,Task"` merge that produced the duplicate-tag error, without affecting any standalone event. The `value.Levels` block was left fully intact (all 107 level definitions preserved), so the human-readable code-to-label mapping is unchanged.

**Remaining warnings (37), left on purpose**
- These are all "recommended but missing" fields that need information from the study, lab, or equipment that isn't in the dataset: manufacturer, manufacturer's model name, software version, device serial number, task description, instructions, cognitive-atlas and CogPO IDs, institution name and address and department, cap manufacturer and model, EEG ground location, head circumference, hardware filters, subject artefact description, and stimulus-presentation software. One HED warning notes that event value `1` is not described in any sidecar; this is expected, since its HED entry was intentionally dropped above to fix the duplicate-tag error. `GeneratedBy` is also listed among the recommended-but-missing fields; it was deliberately left absent rather than filled with a guess.
