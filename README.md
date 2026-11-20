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
- [Missing value handling, e.g. forward fill / interpolation]
- [Scaling, e.g. Min-Max normalization fit on training data only]
- Sliding windows: the previous **[N]** time steps are used to predict the next **[M]** step(s)

### 2. Model
| Component | Setting |
|---|---|
| Architecture | [e.g. 2-layer LSTM, 64 hidden units, dropout 0.2, dense output] |
| Loss | [e.g. MSE] |
| Optimizer | [e.g. Adam, lr = 0.001] |
| Framework | [PyTorch / TensorFlow / Flower / TFF] |

### 3. Federated setup
| Setting | Value |
|---|---|
| Number of clients | [e.g. 5 simulated households] |
| How data was partitioned | [e.g. by household / by time period / randomly] |
| Aggregation | [e.g. FedAvg] |
| Communication rounds | [e.g. 50] |
| Local epochs per round | [e.g. 2] |
| Clients per round | [e.g. all / fraction] |

---

## Results

| Model | RMSE ↓ | MAE ↓ | [MAPE ↓] |
|---|---|---|---|
| Naive baseline (previous value) | [ ] | [ ] | [ ] |
| Centralized LSTM | [ ] | [ ] | [ ] |
| Federated LSTM (FedAvg) | [ ] | [ ] | [ ] |

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
