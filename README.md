# Predicting AMR Patterns in Kazakhstan Using Zero-Shot Machine Learning

Kazakhstan has no entries in any of the three major global AMR surveillance databases (ATLAS, SIDERO-WT, KEYSTONE). This project asks: can you predict resistance rates for a country that was never in the training data?

The answer, it turns out, is mostly yes: if you describe countries by what you *can* measure globally (GDP, healthcare spending, antibiotic consumption by class) rather than by name.

**LightGBM, trained on 12.3M isolates from 83 countries, achieves AUC 0.880 on held-out 2021–2022 data and mean AUC 0.863 across four leave-one-country-out folds.**

---

## How it works

Every country in the training set is represented by macroeconomic proxies: not its name, not its flag, not its geographic region. The model never sees "Kazakhstan" during training. When we run inference, we feed it Kazakhstan's World Bank indicators and antibiotic consumption data and let it generalize.

The domain split was derived by clustering all countries on their macro profiles (k=6). Kazakhstan clusters with only one other country (Qatar), confirming it sits in a genuinely distinct epidemiological space.

---

## Results

| Model | AUC | AUPRC |
|---|---|---|
| Logistic Regression | 0.6955 | 0.3832 |
| Random Forest | 0.8626 | 0.6592 |
| XGBoost | 0.8725 | 0.6932 |
| LightGBM | **0.8797** | **0.6950** |
| CatBoost | 0.8783 | 0.6948 |
| MLP | 0.8580 | 0.6528 |

**LOCO validation** (train on all countries except one, evaluate on the held-out country):

| Held-out country | AUC | AUPRC | n |
|---|---|---|---|
| Bulgaria | 0.8812 | 0.8019 | 23,655 |
| Ukraine | 0.8540 | 0.7997 | 35,440 |
| Turkey | 0.8603 | 0.7024 | 205,543 |
| Mexico | 0.8561 | 0.6891 | 272,056 |
| **Mean** | **0.863 ± 0.013** | **0.748 ± 0.061** | |

**5-fold time-aware CV:** mean AUC 0.881 ± 0.012

---

## Top predicted resistance risks in Kazakhstan (2017–2023)

*Klebsiella pneumoniae* + ampicillin (0.90), *Enterococcus faecium* + amoxicillin-clavulanate (0.87), *Acinetobacter baumannii* + ampicillin (0.85). Full predictions in `outputs/kz_top_resistance_risks_matched.csv`.

---

## Data

You need to obtain the surveillance data yourself — all three databases require a free data access request.

| File | Where to get it |
|---|---|
| `data/atlas.csv` | Request at [vivli.org](https://vivli.org) → ATLAS (Pfizer) |
| `data/sidero_wt.xlsx` | Request at [vivli.org](https://vivli.org) → SIDERO-WT (Shionogi) |
| `data/keystone.xlsx` | Request at [vivli.org](https://vivli.org) → KEYSTONE |
| `data/worldbank/` | [World Bank Open Data](https://data.worldbank.org) — indicators NY.GDP.PCAP.CD, SH.XPD.CHEX.GD.ZS, EN.POP.DNST |
| `data/worldbank-kz/` | Same World Bank indicators, Kazakhstan only |
| `data/antibiotic-consumption-rate.csv` | [Our World in Data](https://ourworldindata.org/antibiotic-resistance) |
| `data/antibiotic_consumption_clean.csv` | [Klein et al. 2018](https://doi.org/10.1016/S1473-3099(18)30605-4) — global antibiotic consumption by class |
| `data/kazakhstan_antibiotics_did_2017_2023.csv` | Included in this repo — compiled from Kazakhstan national AMR reports |

Vivli access is free for research purposes. Approval usually takes 1–2 weeks.

---

## Reproducing

```bash
pip install -r requirements.txt
jupyter notebook notebook.ipynb
```

Run cells in order. The notebook is self-contained — outputs are written to `outputs/`.

Two cells take a while: data harmonization (~10 min on first run, saves `combined_dataset.csv`) and model training (~15–20 min on GPU).

---

## Limitations

The model has no Kazakhstan clinical isolate data to validate against — there isn't any in the public domain. The predictions are the output of a model that has never seen a Kazakhstani patient. LOCO results give us confidence the zero-shot transfer works, but they're proxies, not proof.

The 14 unmatched antibiotics (furazidin, nitroxoline, sulfasodimidine, etc.) are absent from all three surveillance databases — they're regionally specific drugs common in Central Asian clinical practice. Predictions for those antibiotics rely on organism + macro features only.

---


---

## License

Code: MIT. Data files in `data/` are subject to their original source terms — see each database's data use agreement.
