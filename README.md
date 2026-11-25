# Epochs from events

<!-- brainlife-app:begin (generated from the App manifest; edit the manifest, not this block) -->

> **Extracts epochs around events from continuous MEG/EEG data with MNE-Python mne.Epochs and outputs an epochs .fif file, an epochs plot and an HTML report.**

Reads events from a BIDS-style events.tsv, or detects them on stim_channel or from annotations, maps event codes to condition labels and cuts epochs from tmin to tmax with optional baseline correction. Optionally builds trial metadata to label correct responses (assess_correctness) and keep only correct trials (use_correct). Use it after filtering/cleaning raw MEG/EEG and before averaging or decoding.

| | |
|---|---|
| 📥 **Inputs** | `neuro/meeg/mne/raw` · `neuro/meg/fif-override` (optional) |
| 📤 **Outputs** | `neuro/meeg/mne/epochs` · `report/html` |
| 🐳 **Container** | `brainlifemeeg/mne:1.12.1` |
| ⚙️ **Resources** | 1 CPUs · 20 GB memory · 1.5 h walltime |
| 🔗 **Run it** | [doi.org/10.25663/brainlife.app.613](https://doi.org/10.25663/brainlife.app.613) |

#### 📥 Inputs

| Input | Datatype | Required | Description |
|---|---|---|---|
| `raw` | `neuro/meeg/mne/raw` | yes | Continuous MEG/EEG data in MNE raw .fif format. |
| `events` | `neuro/meg/fif-override` (events) | no | Optional BIDS-style events.tsv with sample and value columns; if absent, events are taken from stim_channel or, failing that, from annotations. |

#### 📤 Outputs

| Output | Datatype | Description |
|---|---|---|
| `out_dir` | `neuro/meeg/mne/epochs` | MNE epochs fif file |
| `out_report` | `report/html` | MNE HTML report |

<details>
<summary><b>⚙️ Parameters</b> (12)</summary>

| Parameter | Type | Default | Description |
|---|---|---|---|
| `event_id_condition_mapping` | string |  | Time-locking events, labels, and correctness mapping. Create a mapping between event IDs (1,2,3..., as coded in 'STI 014' or other stimulus aggregate channel) and desired labels (e.g. 'auditory', 'visual', or any other stimulus names that make sense in your design). The mapping should be formatted with dashes (-) between a desired label and the ID code and commas (,) between list entries like so: auditory-1,visual-2, etc. If "assess_correctness" option is toggled, you need to provide additional information about your conditions (event 1) and the correct response (event 2), as well as responses and their types. The information should be formatted as follows: event1/desired_label/(event2)-event_ID Example: stimulus/auditory/left-1: auditory stimulus coded with trigger 1 is supposed to be followed by left stimulus/visual/right-2: visual stimulus coded with trigger 2 is supposed to be followed by right response/left-3: left response coded with trigger 3 response/right-4: right response coded with trigger 4 Several mappings may be separated by commas (,). e.g. stimulus/auditory/left-1, stimulus/visual/right-2, response/left-3:, response/right-4 |
| `tmin` | number | `-0.2` | The time relative to each event at which to start epoch (in seconds). |
| `tmax` | number | `0.5` | The time relative to each event at which to end each epoch (in seconds). |
| `picks` | string | `all` | Channels to include. Slices and lists of integers will be interpreted as channel indices. In lists, channel type strings (e.g., ['meg', 'eeg']) will pick channels of those types, channel name strings (e.g., ['MEG0111', 'MEG2623'] will pick the given channels. Can also be the string values 'all' to pick all channels, or 'data' to pick data channels. None (default) will pick all channels. Note that channels in info['bads'] will be included if their names or indices are explicitly provided. default: 'all' |
| `stim_channel` | string |  | A stim_channel should be present in the data if no events.tsv file is provided as input. |
| `assess_correctness` | boolean | `true` | Assess the number of epochs with correct responses (task-based studies). Dependent upon "event_id_condition_mapping" — need to have stimuli and responses mapped correctly |
| `use_correct` | boolean | `false` | Toggle if want to use only epochs where the participant responded correctly |
| `metadata_tmin` | number | `-0.5` | The startpoint for the epoch in assessing correctness of the response in a trial — can be different from tmin (that is used to actually slice the epochs). |
| `metadata_tmax` | number | `1.1` | The endpoint for the epoch in assessing correctness of the response in a trial — can be different from tmax (that is used to actually slice the epochs). |
| `event1kw` | string | `stimulus` | Event 1 Hierarchical Event Descriptor (HED) keyword (e.g. stimulus). This HED is used to define conditions for epochs. |
| `event2kw` | string | `response` | Event 2 Hierarchical Event Descriptor (HED) keyword (e.g. response). This HED is used to define correct/incorrect epochs. |
| `baseline` | string |  | The time interval to consider as “baseline” when applying baseline correction. If None, do not apply baseline correction. If a tuple (a, b), the interval is between a and b (in seconds), including the endpoints. If a is None, the beginning of the data is used; and if b is None, it is set to the end of the data. If (None, None), the entire time interval is used. |

</details>

📚 **References used by this App:**
- Hayashi S, Caron BA, Heinsfeld AS, et al. brainlife.io: a decentralized and open-source cloud platform to support neuroscience research. Nat Methods 21:809-813 (2024). [doi:10.1038/s41592-024-02237-2](https://doi.org/10.1038/s41592-024-02237-2)
- Gramfort A, et al. MEG and EEG data analysis with MNE-Python. Front Neurosci 7:267 (2013). [doi:10.3389/fnins.2013.00267](https://doi.org/10.3389/fnins.2013.00267)

<!-- brainlife-app:end -->

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
