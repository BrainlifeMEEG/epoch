# Epochs from events

[![Run on Brainlife.io](https://img.shields.io/badge/Brainlife-bl.app.613-blue.svg)](https://doi.org/10.25663/brainlife.app.613)

## Description

This app extracts epochs (time-locked segments) from raw MEG/EEG data around event markers, using MNE-Python's [`mne.Epochs`](https://mne.tools/stable/generated/mne.Epochs.html). Events are read from a provided `events.tsv` input if available; otherwise they are detected from a stimulus channel with [`mne.find_events`](https://mne.tools/stable/generated/mne.find_events.html) or, if no stimulus channel is given either, from annotations with [`mne.events_from_annotations`](https://mne.tools/stable/generated/mne.events_from_annotations.html). Event codes are mapped to condition labels through `event_id_condition_mapping`, and the app can optionally build per-trial metadata with [`mne.epochs.make_metadata`](https://mne.tools/stable/generated/mne.epochs.make_metadata.html) to assess whether a behavioral response matched the expected target.

The app generates:
- Epoched MEG/EEG data (`mne.Epochs`)
- An HTML report with epoch statistics and visualizations
- A plot of the epochs
- `product.json` Brainlife.io metadata, including the epochs plot image

## Inputs

- **`raw`** (`neuro/meeg/mne/raw`): continuous MEG/EEG data to epoch (required)
- **`events`** (`neuro/meg/fif-override`, tag `events`): BIDS-style `events.tsv` with `sample`/`value` columns (optional). If not provided, events are detected from `stim_channel` or, if that is also unset, from annotations in the raw data.

## Outputs

- **`out_dir/meg-epo.fif`** (`neuro/meeg/mne/epochs`): epoched data
- **`out_report/report.html`** (`report/html`): HTML report with epoch visualization and, if `assess_correctness` is enabled, response-correctness counts
- **`out_figs/epochs_plot.png`** (`generic/image/png`): image plot of the epochs (global field power), also embedded in `product.json`

## Configuration Parameters

| key | type | default | description |
|---|---|---|---|
| `event_id_condition_mapping` | string | required | Mapping from numeric event codes to condition labels, formatted `type/label[/category]-ID`, comma-separated (e.g. `stimulus/auditory/left-1,stimulus/visual/right-2,response/left-3,response/right-4`). Parsed into the `event_id` dict passed to `mne.Epochs()`. |
| `tmin` | float | required | Start of the epoch relative to each event, in seconds (passed to `mne.Epochs(tmin=...)`, e.g. `-0.5`). |
| `tmax` | float | required | End of the epoch relative to each event, in seconds (passed to `mne.Epochs(tmax=...)`, e.g. `1.1`). |
| `picks` | string | `"all"` | Channels to include, passed to `mne.Epochs(picks=...)`. `"all"` or `"data"` pick all/data channels; a comma-separated list of channel types or names picks only those; empty/unset is treated as `"all"`. |
| `stim_channel` | string | `""` | Name of the stimulus channel to detect events from with `mne.find_events()` (e.g. `"STI101"`). Only used when the `events` input is not provided; if also empty, events are instead taken from annotations in the raw data via `mne.events_from_annotations()`. |
| `assess_correctness` | boolean | `false` | If `true`, build trial metadata with `mne.epochs.make_metadata()` and add a `<event2kw>_correct` column recording whether each response matched the expected target. Requires `metadata_tmin`/`metadata_tmax`. |
| `use_correct` | boolean | `false` | If `true` (and `assess_correctness` is also `true`), keep only epochs whose response was correct. |
| `metadata_tmin` | float | required if `assess_correctness` is `true` | Start of the window (relative to each event) used by `mne.epochs.make_metadata()` to build trial metadata; may differ from `tmin`. |
| `metadata_tmax` | float | required if `assess_correctness` is `true` | End of the window (relative to each event) used by `mne.epochs.make_metadata()` to build trial metadata; may differ from `tmax`. |
| `event1kw` | string | `"stimulus"` (required) | Hierarchical Event Descriptor (HED) keyword identifying "event 1" (e.g. the stimulus) labels in `event_id_condition_mapping`; used to build metadata and assess correctness. |
| `event2kw` | string | `"response"` (required) | HED keyword identifying "event 2" (e.g. the response) labels in `event_id_condition_mapping`; used to build metadata and assess correctness. |
| `baseline` | string | `""` | Baseline correction window passed to `mne.Epochs(baseline=...)`: a `"(a, b)"` string (`none`/`tmin`/`tmax` allowed for either bound), `"none"` for no correction, or empty for MNE's default `(None, 0)`. |

### Event ID Condition Mapping Format

Format: `type/label[/category]-ID`, comma-separated.

**Examples:**
- Simple: `stimulus/auditory-1,stimulus/visual-2,response/left-3,response/right-4`
- With targets (for `assess_correctness`): `stimulus/D_REA/target_right-13,stimulus/REA_D/target_left-14,response/left-25,response/right-26`

## Usage

### Running on Brainlife.io

1. Select your raw MEG/EEG dataset as the `raw` input, and optionally an `events.tsv` as the `events` input.
2. Set `event_id_condition_mapping`, `tmin`, `tmax`, and the other configuration parameters as needed.
3. Submit the process.
4. Review the epochs plot and HTML report in the output viewer.

### Local Testing

```bash
# Edit config.json to point "raw" (and optionally "events") at real files, then:
python main.py
```

## Technical Details

### Event Detection and Metadata
- Events are read from an `events.tsv` input if provided, otherwise detected from a stimulus channel or from annotations in the raw data.
- Metadata is created using MNE's [`make_metadata`](https://mne.tools/stable/generated/mne.epochs.make_metadata.html) function, only when `assess_correctness` is enabled.
- Behavioral responses can be aligned with stimulus information for accuracy analysis.

### Response Correctness Assessment
When `assess_correctness` is enabled:
1. Stimulus and response events are mapped to target categories.
2. Response correctness is determined by matching response type with stimulus target.
3. A `<event2kw>_correct` column is added to epoch metadata.
4. If `use_correct` is `true`, only correct-response epochs are kept.

### Output Report
The HTML report includes:
- An interactive display of the epochs
- Correct/incorrect response counts, if `assess_correctness` is enabled

## Authors
- Kami Salibayeva (https://github.com/KSalibay)
- Maximilien Chaumon (https://github.com/dnacombo), Paris Brain Institute

## Citations

- Hayashi, S., Caron, B.A., Heinsfeld, A.S. et al. brainlife.io: a decentralized and open-source cloud platform to support neuroscience research. Nat Methods 21, 809–813 (2024). https://doi.org/10.1038/s41592-024-02237-2
- Gramfort, A. et al. MEG and EEG data analysis with MNE-Python. Front. Neurosci. 7, 267 (2013). https://doi.org/10.3389/fnins.2013.00267

## Funding Acknowledgement

brainlife.io is publicly funded and for the sustainability of the project it is helpful to acknowledge the use of the platform. We kindly ask that you acknowledge the funding below in your code and publications.

[![NSF-BCS-1734853](https://img.shields.io/badge/NSF_BCS-1734853-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1734853)
[![NSF-BCS-1636893](https://img.shields.io/badge/NSF_BCS-1636893-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1636893)
[![NSF-ACI-1916518](https://img.shields.io/badge/NSF_ACI-1916518-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1916518)
[![NSF-IIS-1912270](https://img.shields.io/badge/NSF_IIS-1912270-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1912270)
[![NIH-NIBIB-R01EB029272](https://img.shields.io/badge/NIH_NIBIB-R01EB029272-green.svg)](https://grantome.com/grant/NIH/R01-EB029272-01)
[![NIH-NIBIB-R01EB030896](https://img.shields.io/badge/NIH_NIBIB-R01EB030896-green.svg)](https://grantome.com/grant/NIH/R01-EB030896-01)

## License

Copyright (c) 2026 MEEG Brainlife team. Licensed under AGPL-3.0, see [license.txt](license.txt).
