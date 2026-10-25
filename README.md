# QoE prediction on the PoQeMoN dataset

Course project for **Data Learning: From Theory to Practice** (Télécom SudParis, 2026).

**Authors:** Misato Kanayama, Yasmin [family name]

We compare machine learning models that predict the **quality of experience (QoE)** of mobile video sessions.
The target is the **MOS** (Mean Opinion Score), a rating from 1 (bad) to 5 (excellent) given by the user after each video.

---

## 1. Get the data

The data comes from the PoQeMoN QoE dataset by L. Amour, S. Souihi and A. Mellouk:
<https://github.com/Lamyne/Poqemon-QoE-Dataset>

1. Download `pokemon.csv` from that repository.
2. Put it in the **same folder as the notebooks** (the root of this repository).

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
| `02_models.ipynb` *(coming soon)* | Splits the data by user, trains the baseline and the models, runs the grid search | results tables and figures |

No manual changes are needed between notebooks.

---

## 4. Main choices

| Choice | Reason |
|---|---|
| The QoF columns (`QoF_begin`, `QoF_shift`, `QoF_audio`, `QoF_video`) are **not** used in the main experiment | They are the user's own ratings, given at the same time as MOS. They would not be available in a real prediction situation (data leakage). |
| The data is split **by user** | Each user has about 8.5 sessions. Testing on new users gives an honest score. |
| Network type, operator and phone model are **one-hot encoded** | These codes are categories, not quantities. |
| A **baseline** that always predicts the average MOS | Half of the scores are 4, so every model must beat this simple guess. |

---

## 5. Reproducibility

- Random seed: **42** everywhere (`RANDOM_SEED` in each notebook).
- Software versions used: see `requirements.txt`.
- Tested with Python 3.13, pandas 3.0, numpy 2.5, matplotlib 3.11, scikit-learn 1.9.

---

## 6. Repository structure

```
.
├── README.md
├── requirements.txt
├── 01_data_cleaning.ipynb
└── pokemon.csv            ← you add this file (see section 1)
```

---

## 7. Use of generative AI

We used **Claude (Anthropic)** to explain machine learning concepts, to suggest cleaning rules, and to draft code and this README.
We checked the output by running every notebook from start to end, by comparing the results with our own calculations in Google Sheets (MOS counts, correlations, group averages), and by discussing each choice together.

---

## 8. References

- L. Amour, S. Souihi, S. Hoceini, A. Mellouk. *Building a Large Dataset for Model-based QoE Prediction in the Mobile Environment.* ACM MSWiM 2015, pp. 313–317.
- PoQeMoN QoE dataset: <https://github.com/Lamyne/Poqemon-QoE-Dataset>
- F. Pedregosa et al. *Scikit-learn: Machine Learning in Python.* JMLR 12, 2011.
