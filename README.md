# QoE prediction on the PoQeMoN dataset

Course project for **Data Learning: From Theory to Practice** (Télécom SudParis, 2026).

**Authors:** Misato Kanayama, Yasmin Reis

We compare machine learning models that predict the **quality of experience (QoE)** of mobile video sessions.
The target is the **MOS** (Mean Opinion Score), a rating from 1 (bad) to 5 (excellent) given by the user after each video.

---

## 1. Get the data

The data comes from the PoQeMoN QoE dataset by L. Amour, S. Souihi and A. Mellouk:
<https://github.com/Lamyne/Poqemon-QoE-Dataset>

`pokemon.csv` is included in this repository (copied from the original dataset, with credit to the authors). You can also download it from the original repository above and put it in the same folder as the notebooks.

The file has 1543 video sessions, 181 users and 23 columns. It has no missing values.

---

## 2. Install

You need **Python 3.10 or newer**.

```bash
git clone https://github.com/pickle-pori05/poqemon-qoe-prediction.git
cd poqemon-qoe-prediction
pip install -r requirements.txt
```

**Google Colab:** nothing to install. Upload the notebook with *File → Upload notebook*. When the first cell asks for it, upload `pokemon.csv`.

---

## 3. Run the analysis

Run the notebooks **in this order**, each one from top to bottom (*Run All*):

| Notebook | What it does | Output |
|---|---|---|
| `01_data_cleaning.ipynb` | Removes errors and duplicates, applies a log to buffering time, groups rare phone models, defines the feature groups | `pokemon_clean.csv` |
| `03_models.ipynb` | Shows why a single 80/20 split is unstable, then compares the models with **nested cross-validation by user** (5 outer folds, grid search inside each fold), with RMSE, MAE, QWK, macro F1, accuracy and the confusion matrix. Also runs the leakage experiment and computes feature importance (about 20 minutes; the random forest grid search is the slowest part) | `results_*.csv`, `fig_*.png` |

A first version of the model notebook (`02_models.ipynb`, one 80/20 split) was replaced by `03_models.ipynb`. It is still available in the Git history.

No manual changes are needed between notebooks. Run `01` first: the other notebooks read `pokemon_clean.csv`.

---

## 4. Project status

| Step | Status |
|---|---|
| Data exploration (Google Sheets) and cleaning (`01`) | ✅ Done |
| First model comparison with a single test split (old `02`, replaced) | ✅ Done |
| Nested cross-validation + new metrics (`03`) | ✅ Done |
| Final model set: baseline, linear regression, random forest, ordinal regression | 🔄 Ordinal regression to do |
| Report and slides | ⬜ To do |

**Main findings so far**
- Buffering time and frame rate are the most useful features.
- Using the QoF columns lowers the error a lot, which confirms the data leakage.
- With the same random forest, 10 different 80/20 splits give a test MAE from 0.493 to 0.556: the score depends on which 37 users are in the test set. With nested cross-validation, MAE is 0.525 ± 0.03.
- The regression models never predict a 5 (highest prediction 4.29): they are pulled towards the average. RMSE and MAE hide this, so we add QWK, macro F1 and the confusion matrix.

---

## 5. Main choices

| Choice | Reason |
|---|---|
| The QoF columns (`QoF_begin`, `QoF_shift`, `QoF_audio`, `QoF_video`) are **not** used in the main experiment | They are the user's own ratings, given at the same time as MOS. They would not be available in a real prediction situation (data leakage). |
| The data is split **by user** | Each user has about 8.5 sessions. Testing on new users gives an honest score. |
| Network type, operator and phone model are **one-hot encoded** | These codes are categories, not quantities. |
| A **baseline** that always predicts the average MOS | Half of the scores are 4, so every model must beat this simple guess. |
| **Nested cross-validation** by user instead of one test split | With only 37 test users, the score depended on who was in the test set. Nested CV tests every user once and gives a mean ± standard deviation. |
| **QWK, macro F1 and confusion matrix** in addition to RMSE and MAE | MOS is an ordered score. RMSE and MAE hide that the model never predicts 5. |
| Final models: linear regression, random forest, **ordinal regression** | Simple model, non-linear model, and a model made for ordered scores. Random forest and gradient boosting gave the same results, so we kept the simpler one. |

---

## 6. Reproducibility

- Random seed: **42** everywhere (`RANDOM_SEED` in each notebook).
- Data split: `GroupShuffleSplit` by `user_id`, 20% of users in the test set (seed 42) → 1183 training sessions (144 users), 358 test sessions (37 users).
- Cross-validation: `GroupKFold` with 5 folds by `user_id`; grid search chooses the settings with the lowest RMSE.
- Software versions used: see `requirements.txt`.
- Tested with Python 3.13 (Anaconda): pandas 2.3, numpy 2.3, matplotlib 3.10, scikit-learn 1.7.

---

## 7. Repository structure

```
.
├── README.md
├── requirements.txt
├── .gitignore                 ← pokemon_clean.csv is not uploaded (notebook 01 recreates it)
├── pokemon.csv                ← original data
├── 01_data_cleaning.ipynb
├── 03_models.ipynb
├── fig_*.png                  ← figures made by notebook 03
└── results_*.csv              ← result tables made by notebook 03
```

---

## 8. Use of generative AI

We used **Claude (Anthropic)** to explain machine learning concepts, to suggest cleaning rules, and to draft code and this README.
We checked the output by running every notebook from start to end, by comparing the results with our own calculations in Google Sheets (MOS counts, correlations, group averages), and by discussing each choice together.

*To complete at the end of the project with each member's tools and tasks.*

---

## 9. References

- L. Amour, S. Souihi, S. Hoceini, A. Mellouk. *Building a Large Dataset for Model-based QoE Prediction in the Mobile Environment.* ACM MSWiM 2015, pp. 313–317.
- PoQeMoN QoE dataset: <https://github.com/Lamyne/Poqemon-QoE-Dataset>
- F. Pedregosa et al. *Scikit-learn: Machine Learning in Python.* JMLR 12, 2011.
