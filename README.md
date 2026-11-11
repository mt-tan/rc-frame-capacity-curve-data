# Data and trained models

This repository contains the data, the trained surrogate models and the Supplementary Material of the article

> M. Tan, A. Pala, M.T. Göğüş, Capacity-curve surrogate for planar reinforced concrete frames: accuracy and preservation of the stirrup-spacing effect.

The frames are planar reinforced concrete frames analyzed by pushover analysis in OpenSeesPy. The limits and the
seismic demand follow the Turkish Building Earthquake Code (TBEC 2018). Symbols, definitions and methods are given
in the article and in `Supplementary_Material.pdf`.

## Contents

| Path | Content |
|---|---|
| `data/frames_main.csv` | Main population, 29,200 frames, one row per frame: the 34 surrogate inputs, modal quantities, yield and peak points, event drifts with censoring indicators, shear events and target displacements. Column `set` separates the training pool (20,432) from the hold-out set (8,768). |
| `data/frames_sensitivity.csv` | Sensitivity population, 11,400 frames, same columns. |
| `data/holdout_predictions.csv` | Surrogate predictions for the hold-out frames and the target displacements derived from them. |
| `data/holdout_curves_recorded.csv`, `data/holdout_curves_predicted.csv` | Recorded and predicted capacity curves of the hold-out frames at 160 roof drift ratios. |
| `data/training_cv_predictions.csv` | Out-of-fold predictions of the five-fold cross-validation on the training pool. |
| `data/single_option_test.csv` | Recorded and predicted changes in the re-analyses of ten hold-out frames with other stirrup options. |
| `data/column_tests.csv` | Records of the PEER Structural Performance Database selected for the check of the column model (85 records; 82 used, which form 73 specimens). |
| `data/table_S4_sensitivity_population.csv` | Supplementary Table S4 for the sensitivity population. |
| `figure_data/` | Numerical data of Figs. 1(b) and 2-8 of the article. Fig. 9 is drawn from `data/single_option_test.csv`. |
| `models/` | Trained CatBoost models of the surrogate. |
| `data_dictionary.csv` | Description and unit of every column. |
| `Supplementary_Material.pdf` | Supplementary Material of the article. |
| `LICENSE` | License terms. The data and the trained models are licensed under CC BY 4.0. |

The frame, prediction and curve files are linked by `frame_id`. The rows of the two curve files and of
`holdout_predictions.csv` are in the same order.

## Conventions

- Event drifts and target displacements are roof displacements divided by the total height (`height_total_m`).
- An event that was not reached before the first bar rupture or the end of the analysis is right-censored:
  `censored_<event>` is 1 and `drift_<event>` is the drift at which the observation ended. Censored drifts are not
  event drifts and must be treated as lower bounds (the article uses the Kaplan-Meier method).
- Spacing groups of the article: 5-8, 9-12, 13-16 and 17-20 cm, by `column_stirrup_spacing_m` for the column events
  and by `beam_stirrup_spacing_m` for the beam events.
- Hazard levels: 475 (475-year) and 2475 (2475-year); soil classes ZA to ZE.
- Empty cells mean "not defined" (for example, no bar rupture, or an analysis that ended before a grid point).

## Trained models

The models were trained on the training pool only. Each model takes the 34 inputs in the order below, in the units
of `data/frames_main.csv`, and returns the natural logarithm of its target.

| Model file | Target | Unit | Training frames |
|---|---|---|---|
| `T1.cbm` | first-mode period | s | 20,432 |
| `Vy.cbm` | yield base shear of the idealized curve | kN | 12,213 |
| `drift_y.cbm` | yield roof drift ratio of the idealized curve | - | 12,213 |
| `Vmax.cbm` | peak base shear | kN | 12,213 |
| `drift_IO_b.cbm` | roof drift ratio at the beam IO event | - | 20,260 |
| `drift_LS_b.cbm` | roof drift ratio at the beam LS event | - | 19,805 |
| `drift_CP_b.cbm` | roof drift ratio at the beam CP event | - | 19,505 |
| `drift_IO_c.cbm` | roof drift ratio at the column IO event | - | 18,566 |
| `drift_LS_c.cbm` | roof drift ratio at the column LS event | - | 17,015 |
| `drift_CP_c.cbm` | roof drift ratio at the column CP event | - | 15,861 |
| `VVr_at_LS_b.cbm` | beam shear ratio V/Vr at the beam LS event | - | 19,805 |
| `Gamma1.cbm` | first-mode participation factor (mode normalized at the roof) | - | 20,432 |
| `mass_participation.cbm` | effective mass ratio of the first mode | - | 20,432 |
| `curve.cbm` | base shear at a given roof drift ratio | kN | 20,432 |

The event-drift models were trained on the frames that reached the event, and the models of `Vy`, `drift_y` and
`Vmax` on the frames whose curve passed a peak. `curve.cbm` takes 35 inputs, the 34 inputs followed by the roof drift
ratio, and returns the logarithm of the base shear at that drift; it was trained on the curves up to the first bar
rupture, on the grid of 160 drift ratios from 0.0005 to 0.08. The target displacement is not a model output. It is
derived from the predicted curve, `T1`, `Gamma1` and M1* = predicted mass participation x `mass_total_t` with the
procedure of TBEC 2018 (Sections 2.3 and 2.5 of the article).

| Input | Column | Unit |
|---|---|---|
| 1 | `n_stories` | - |
| 2 | `n_bays` | - |
| 3 | `span_mean_m` | m |
| 4 | `span_max_m` | m |
| 5 | `span_min_m` | m |
| 6 | `story_height_first_m` | m |
| 7 | `story_height_typical_m` | m |
| 8 | `column_width_m` | m |
| 9 | `beam_width_m` | m |
| 10 | `beam_depth_m` | m |
| 11 | `cover_m` | m |
| 12 | `fck_kPa` | kPa |
| 13 | `fyk_MPa` | MPa |
| 14 | `fsu_MPa` | MPa |
| 15 | `eps_sy` | - |
| 16 | `steel_hardening_ratio` | - |
| 17 | `beam_dead_load_kN_m` | kN/m |
| 18 | `beam_live_load_kN_m` | kN/m |
| 19 | `frame_width_m` | m |
| 20 | `column_gravity_force_kN` | kN |
| 21 | `axial_load_ratio` | - |
| 22 | `column_long_ratio` | - |
| 23 | `column_bar_diameter_m` | m |
| 24 | `column_bars_per_face` | - |
| 25 | `beam_top_ratio` | - |
| 26 | `beam_bottom_ratio` | - |
| 27 | `beam_bar_diameter_m` | m |
| 28 | `beam_top_bars` | - |
| 29 | `beam_bottom_bars` | - |
| 30 | `column_stirrup_diameter_m` | m |
| 31 | `column_stirrup_spacing_m` | m |
| 32 | `column_stirrup_legs` | - |
| 33 | `beam_stirrup_diameter_m` | m |
| 34 | `beam_stirrup_spacing_m` | m |

Example (Python, `catboost` and `pandas`):

```python
import numpy as np, pandas as pd
from catboost import CatBoostRegressor

frames = pd.read_csv('data/frames_main.csv')
X = frames[frames.set == 'holdout'].iloc[:, 2:36].to_numpy(float)    # the 34 inputs, in order

model = CatBoostRegressor().load_model('models/drift_LS_c.cbm')
drift_LS_c = np.exp(model.predict(X))                                # roof drift ratio at the column LS event

curve = CatBoostRegressor().load_model('models/curve.cbm')
grid = np.round(np.arange(1, 161) * 0.0005, 4)
V = np.exp(curve.predict(np.column_stack([np.repeat(X[:1], 160, axis=0), grid])))   # kN, first frame
```

The surrogate is valid within the sampled ranges of the inputs (Table 2 of the article) and for the modeling
assumptions stated there. A final detailing should be confirmed by nonlinear analysis.

## Not included

The capacity curves of the training pool and of the sensitivity population, the analysis scripts and the
pre-registration records are available from the corresponding author on request.
