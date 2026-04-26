# Visuotactile RPDF Dataset

This repository contains the active dataset used by the RPDF visuo-proprioceptive
object-property prediction experiments.

Code repository:
https://github.com/Jiaming-Zhu/visuotactile

## Introduction

The dataset records robot-object interactions for a low-cost physical-property
prediction project. Each sample combines a pre-contact RGB image with internal
motor feedback from the robot, allowing the companion software to evaluate
whether proprioceptive evidence can reduce shortcut failures under deceptive
visual cues.

The repository is separated from the software repository so that the code can
stay lightweight while the data remain publicly accessible and versioned at the
point of final-report submission.

## Contextual Overview

```text
physical object
  -> SO-101 grasp-like interaction
  -> visual_anchor.jpg + tactile_data.pkl + metadata.json
  -> train/val/test/ood_test split
  -> RPDF training and evaluation code
  -> mass, stiffness, and material predictions
```

## Task

Each episode is one robot-object interaction. The learning task is to predict
three physical properties from a pre-contact RGB image and a proprioceptive
time-series signal collected from the robot motors.

| Property | Classes |
| --- | --- |
| Mass | `low`, `medium`, `high` |
| Stiffness | `soft`, `medium`, `rigid` |
| Material | `sponge`, `foam`, `wood`, `container` |

## Dataset Size

| Split | Episodes |
| --- | ---: |
| `train` | 607 |
| `val` | 76 |
| `test` | 76 |
| `ood_test` | 150 |
| **Total** | **909** |

The full directory is about 728 MB. The largest individual file is below 1 MB,
so this repository does not require Git LFS.

## Installation and Access

Clone the dataset repository:

```bash
git clone https://github.com/Jiaming-Zhu/visuotactile-rpdf-dataset.git
```

No Python package installation is required for the dataset itself. To train or
evaluate models, install the dependencies listed in the companion code
repository README and pass this dataset directory as `--data_root`.

## Structure

```text
Plaintextdataset/
├── train/
│   ├── physical_properties.json
│   └── <object_name>/
│       └── episode_YYYYMMDD_HHMMSS_xxxxxx/
│           ├── metadata.json
│           ├── tactile_data.pkl
│           └── visual_anchor.jpg
├── val/
├── test/
└── ood_test/
```

`physical_properties.json` is the authoritative label source for each split.
Per-episode metadata are retained for traceability, but downstream training code
uses the object directory and split-level physical-property manifest for labels.

## Modalities

- `visual_anchor.jpg`: static RGB image before or at the start of the grasp.
- `tactile_data.pkl`: 24-channel proprioceptive sequence.
- `metadata.json`: episode-level recording metadata.

The 24 proprioceptive channels are:

| Channels | Signal |
| --- | --- |
| `0-5` | joint position |
| `6-11` | joint load |
| `12-17` | joint current |
| `18-23` | joint velocity |

## Splits

The `train`, `val`, and `test` splits cover the in-distribution object set. The
`ood_test` split contains deceptive-object cases used to test shortcut
robustness under misleading visual cues.

Objects in `ood_test`:

- `Box_Rock_Blue`
- `Cardbox_hollow_noise`
- `Sponge_Red`
- `WoodBlock_noise1`
- `YogaBrick_Foil_ANCHOR`

## Use With the Code Repository

Clone this dataset next to the code repository or pass its path explicitly:

```bash
python scripts/train_fusion_gating_online_reliable.py \
  --mode train \
  --data_root /path/to/Plaintextdataset \
  --save_dir outputs/fusion_gating_online_reliable_seed42 \
  --seed 42 \
  --device cuda \
  --epochs 50 \
  --visual_mismatch_prob 0.25 \
  --lambda_mismatch_gate 0.2 \
  --reliable_selection_start_epoch 16 \
  --primary_checkpoint reliable \
  --no_live_plot
```

## Notes

This dataset is intended for academic research and reproducibility of the RPDF
experiments. Check the code repository for model definitions, evaluation
scripts, and paper-facing results.

## Known Issues and Future Improvements

- The dataset is controlled rather than a large-scale natural benchmark. It is
  designed to expose deceptive visual-cue failures, not to cover all household
  or industrial objects.
- Data were collected with one robot platform and one grasp-like interaction
  protocol. Cross-robot transfer and richer manipulation actions remain future
  work.
- Labels are discrete property classes. Future datasets could include
  continuous mass, stiffness, compliance, and friction measurements.
- The OOD split focuses on a small set of deceptive objects. Additional object
  identities would improve statistical coverage.

## Third-Party Data and Licences

This repository contains project-collected robot episodes, metadata, and images.
It does not include third-party datasets, model checkpoints, or external source
code. The companion code repository depends on third-party Python and robotics
packages, which remain under their own licences. This dataset is provided as an
academic assessment and reproducibility artefact; contact the author before
redistributing it outside that context.
