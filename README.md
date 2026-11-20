<!--
  WHERE THIS GOES: add as README.md in the Energy-Consumption-Using-LSTM-and-Federated-Learning repo.
  IMPORTANT: everything in [square brackets] must come from YOUR actual experiments.
  Never put in a number you haven't measured. Admissions readers and interviewers may ask about it.
  If you haven't run the centralized-vs-federated comparison yet, that's the single most
  valuable thing to add to this project.
-->

# ⚡ Federated Energy Forecasting with LSTMs

**Can households collaboratively train an energy forecasting model without sharing their raw consumption data?**

This project forecasts household electricity consumption using an LSTM network, and compares a conventional **centralized** model (all data pooled on one server) against a **federated** model (each household trains locally and only shares model updates).

<!-- Optional: put your best forecast plot right here, e.g.  ![Forecast vs actual](figures/forecast.png) -->

---

## Motivation

Smart meters produce fine-grained consumption data that is useful for demand forecasting and grid planning, but it is also sensitive: usage patterns can reveal when people are home, asleep, or away. Federated learning offers a way to train a shared model while keeping each household's data on-device. This project measures how much forecasting accuracy is retained under that constraint.

---

## Dataset

- **Coverage:** household electricity readings from 2016 to 2024, split across two files (`2016–2019`, `2020–2024`)
- **Granularity:** [e.g. hourly / daily]
- **Features used:** [e.g. consumption (kWh), timestamp features such as hour, weekday, month; weather if any]
- **Train / validation / test split:** [e.g. chronological split: 2016–2022 train, 2023 validation, 2024 test]

> Note: splitting chronologically (rather than randomly) avoids leaking future information into training.

---

## Method

### 1. Preprocessing
- **Missing values:** none present. The notebook asserts that the daily series has no gaps.
- **Dropped columns:** the row-counter columns `Day` and `Day.1`, so the model cannot memorise position in time. Also `day_of_week`, which matches the real weekday for only 67% of rows; the weekday is recomputed from `Date`.
- **Scaling:** Min-Max normalization fit on the **training rows only**. Inputs and target have separate scalers. Metrics are reported in kWh after inverse-transforming.
- **Sliding windows:** the previous **14** days are used to predict the next **1** day. Only past days are used as inputs, including past weather.

### 2. Model
| Component | Setting |
|---|---|
| Architecture | 1-layer LSTM, 32 hidden units, linear output |
| Loss | MSE |
| Optimizer | Adam, lr = 0.001, batch size 32 |
| Model selection | Early stopping on validation loss (patience 40, max 500 epochs) |
| Framework | PyTorch, with FedAvg implemented from scratch |

### 3. Federated setup
| Setting | Value |
|---|---|
| Number of clients | 3 simulated clients |
| How data was partitioned | By time: the training period is split into three contiguous segments of ~300 samples each. Each segment covers different seasons, so the clients are non-IID. |
| Aggregation | FedAvg: every parameter is averaged, weighted by client sample count |
| Communication rounds | Up to 300, keeping the round with the best validation loss (patience 40) |
| Local epochs per round | 2 |
| Clients per round | All |

### Baselines
- **Naive:** predicts yesterday's value.
- **Seasonal naive:** predicts the value from 7 days earlier.
- **Local:** one LSTM per client, trained on that client's data only.
- **Centralized:** one LSTM trained on all training data pooled.

---

### Key findings
- [e.g. The federated model reached RMSE of X vs Y for the centralized model, retaining most of the accuracy without sharing raw data.]
- [e.g. Convergence required ~Z rounds; performance plateaued after that.]
- [e.g. Clients with unusual consumption patterns had higher error, suggesting non-IID data hurts the shared model.]

---

## Limitations

Being explicit about what this project does *not* show:

- **Simulated federation:** clients run on a single machine, so real-world issues like network latency, dropouts and stragglers are not modeled.
- **No formal privacy guarantee:** model updates can still leak information. Techniques like differential privacy or secure aggregation were not applied.
- **[Non-IID data:** households differ in size and habits; the effect of this heterogeneity was [only partially / not] studied.]
- **[Limited number of clients:** results may not generalize to deployments with thousands of households.]

---

## Future work

- Add **differential privacy** (e.g. DP-SGD) and measure the privacy–accuracy trade-off
- Try aggregation methods designed for heterogeneous clients, such as **FedProx**
- Compare the LSTM against a **Transformer-based** forecaster
- Personalization: fine-tune the global model locally for each household

---

## Repository structure

```
├── data/
│   ├── Household_electricity_dataset 2016-2019.csv
│   └── Household_electricity_dataset 2020-2024.csv
├── src/
│   ├── data.py          # loading, cleaning, windowing
│   ├── model.py         # LSTM definition
│   ├── centralized.py   # centralized training baseline
│   └── federated.py     # client/server simulation and FedAvg
├── notebooks/
│   └── Energy Prediction NB.ipynb   # exploration and result plots
├── figures/
├── requirements.txt
└── README.md
```
<!-- This is the RECOMMENDED structure. Either reorganize to match, or edit this block to match what you have. -->

---

## How to run

```bash
git clone https://github.com/samhithanair/Energy-Consumption-Using-LSTM-and-Federated-Learning.git
cd Energy-Consumption-Using-LSTM-and-Federated-Learning
pip install -r requirements.txt

# Option A: run the notebook
jupyter notebook "notebooks/Energy Prediction NB.ipynb"

# Option B (once code is modularized)
python src/centralized.py
python src/federated.py --clients 5 --rounds 50
```

Random seeds are fixed at **[42]** for reproducibility.

---

## Author

**Samhitha Nair** · [LinkedIn](https://www.linkedin.com/in/samhithanair/) · [GitHub](https://github.com/samhithanair)
