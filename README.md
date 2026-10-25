# ☀️ SolarVision

**SolarVision** is a decision-support system for solar PV plant operations.  
It helps operators forecast PV power, detect abnormal production drops, decide whether panel cleaning is economically justified, and prioritize panels that may need inspection using infrared image analysis.

> **Note:** This project uses open datasets as substitutes for real Egyptian plant data. Financial assumptions are user-defined and results are intended for decision support, not guaranteed real-world savings.

---

## 🎯 Problem

Dust accumulation reduces solar panel output. Operators need to know:

- What production is expected in the next 24–48 hours?
- Is actual production abnormally low?
- Is cleaning economically justified?
- Which panels should be inspected first?

---

## 🚀 Features

- **PV power forecasting** for the next 24 hours using historical production, dust, calendar, and weather features.
- **Anomaly detection** by comparing actual power with forecasted power.
- **Cleaning decision support** using expected recoverable energy, electricity price, and user-defined cleaning cost.
- **Infrared image classification** for PV module condition using a 12-class deep learning model.
- **Model explainability** using SHAP.
- **Experiment tracking** using MLflow and hyperparameter tuning using Optuna.
- **FastAPI backend**, **Streamlit dashboard**, and **Docker deployment**.

---

## 📊 Data

### 1. PV Power and Dust Dataset

- Source: [Hourly PV Power and Dust Accumulation Dataset — Laghouat, Algeria](https://data.mendeley.com/datasets/4jdt83yy43/3)
- Period: February–August 2022.
- Frequency: Hourly.
- Records: 4,368.
- Contains clean and dusty PV module power, dust density, irradiance, temperature, humidity, wind, cloud cover, and precipitation.

**Limitations:** the dataset covers around six months, represents two small PV modules rather than a utility-scale plant, and dust density was reconstructed hourly from limited measurements.

### 2. Infrared Solar Modules Dataset

- Source: [InfraredSolarModules — Raptor Maps](https://github.com/raptormaps/InfraredSolarModules)
- Contains 20,000 infrared images of PV modules.
- Includes 12 classes: one normal class and 11 anomaly/fault classes.
- Used for the computer-vision module that classifies panel thermal images and supports maintenance prioritization.

---

## 🧠 AI Modules

### 1. Power Forecasting

The forecasting model predicts dusty PV module power 24 hours ahead.

Feature groups include:

- Calendar and cyclical time features.
- Historical production lags and rolling statistics.
- Dust density and dust-history features.
- Weather features such as temperature, humidity, wind, cloud cover, and precipitation.

The data was split chronologically to prevent future information leakage:

| Split | Percentage |
|---|---:|
| Train | 70% |
| Validation | 15% |
| Test | 15% |

Three model families were compared:

| Family | Model |
|---|---|
| Linear | Ridge Regression |
| Tree ensemble | Random Forest |
| Gradient boosting | LightGBM |

LightGBM was tuned using Optuna with time-series cross-validation and tracked using MLflow.

Models were evaluated using MAE, RMSE, daytime-only MAPE, and R².

### 2. Anomaly Detection

The system compares actual PV power with forecasted PV power.

An alert is raised when actual power remains significantly below the forecast for multiple hours. This reduces false alerts during low-irradiance morning and evening periods.

The anomaly module is a warning system only. It does not claim to distinguish between soiling, shading, inverter faults, or other equipment issues.

### 3. Cleaning Decision

The cleaning recommendation compares estimated recoverable energy with the user-defined cleaning cost:

```text
Estimated Loss = Expected Energy - Actual Energy
Recovered Energy = Estimated Loss × Recoverable Fraction
Revenue Gain = Recovered Energy × Electricity Price
Net Benefit = Revenue Gain - Cleaning Cost
```

The system recommends:

- **Clean** when net benefit is positive.
- **Wait** when net benefit is zero or negative.

### 4. Infrared Image Classification

The computer-vision module classifies infrared images of PV modules into 12 conditions, including normal operation and fault/anomaly categories.

The output includes:

- Predicted module condition.
- Model confidence.
- Visual explanation of important image regions.
- A maintenance priority to help technicians decide which panels to inspect first.

The image module is independent from the power-forecasting module. It supports inspection prioritization but does not claim to explain energy losses.

---

## 🏗️ Architecture

```text
Streamlit UI
     |
     v
FastAPI + Pydantic
     |
     ├── /health
     ├── /forecast
     ├── /anomaly
     ├── /cleaning-decision
     └── /image-classification
     |
     ├── Time-series forecasting model
     ├── Anomaly detection logic
     ├── Cleaning cost-benefit rule
     └── Infrared image classification model
```

---

## 🛠️ Tech Stack

- Python
- Pandas
- NumPy
- Scikit-learn
- LightGBM
- Optuna
- MLflow
- SHAP
- TensorFlow / PyTorch
- FastAPI
- Streamlit
- Docker

---

## 📁 Project Structure

```text
SolarVision/
│
├── data/
├── notebooks/
├── processed_data/
├── models/
├── app/
│   ├── main.py
│   ├── schemas.py
│   ├── services.py
│   └── dashboard.py
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── README.md
```

---

## ⚠️ Limitations

- The PV dataset is from Algeria and covers only six months.
- It represents small PV modules, not a full solar plant.
- Dust density is partially reconstructed rather than directly measured every hour.
- Weather and irradiance data are derived/reanalysis data.
- The image-classification and power-forecasting modules use separate datasets and are not causally linked.
- Cleaning cost, electricity price, recovery fraction, horizon, and plant scale are user assumptions.
- The system is for decision support and educational demonstration, not safety-critical grid operation.

---

## 🔮 Future Work

- Add a dedicated 48-hour forecasting model.
- Add rolling-origin backtesting and prediction intervals.
- Use live weather forecasts instead of historical weather assumptions.
- Link image-based maintenance priorities with plant-level production data.
- Add drift monitoring and automated retraining.

---

## 👥 Team

| Role | Responsibility |
|---|---|
| Data & EDA | Data audit, EDA, validation labels |
| Forecasting | Feature pipeline, baseline and ML models, tuning |
| Computer Vision | Infrared image classification and explainability |
| Anomaly & Decision | Anomaly detection, cleaning decision, SHAP |
| Deployment | FastAPI, Streamlit, Docker, documentation |
