# Visuotactile RPDF Dataset

This repository contains the active dataset used by the RPDF visuo-proprioceptive
object-property prediction experiments.

Code repository:
https://github.com/Jiaming-Zhu/visuotactile

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
