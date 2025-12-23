# Solax-EMHASS

Automated energy management for a **Qcells/SolaX X3 Hybrid inverter** using **EMHASS (Energy Management for Home Assistant)** and Home Assistant automations.

This repository contains Home Assistant configurations, sensors, MPC (Model Predictive Control) payload templates, and battery management automations that optimize solar usage with price forecasts via EMHASS.

---

## 🧠 Overview

This project integrates:

- **EMHASS** for energy optimization with a 15-minute MPC horizon  
- **Solax Power Hybrid Inverter (X3-Hybrid)** controlled via the Home Assistant `solax` integration  
- **Home Assistant Automations** to execute remote control modes based on MPC forecasts  
- Dynamic scheduling of battery charge/discharge based on price, PV forecast, and load

The goal is to **maximize self-consumption**, **minimize grid cost**, and **keep the battery within safe state-of-charge (SOC) bounds**.

---

## ⚙️ Requirements

This setup assumes:

- **Home Assistant** (Container or OS)  
- **EMHASS** installed and reachable via REST API  
- **Solax Power integration** for the inverter & battery (Modbus or built-in API)  
- Qcells/SolaX inverter with firmware that supports Mode 8 remote control  
- Forecast sources such as Solcast and/or price feeds

---

## 🗂 Repository Structure

Solax-Emhass/
├── Emhass-Battery-Closed-Loop.yml
├── Emhass-Dayahead.yml
├── Emhass-Predict.yml
├── Emhass-Publish.yml
├── Emhass-naive-mpc-forecast.yml
├── sensors.yaml
├── rest_commands.yml
└── README.md

- `sensors.yaml`: Template sensors for forecasts and MPC inputs  
- `rest_commands.yml`: REST definitions for EMHASS API calls  
- `*.yml` automations: MPC control & battery automations

---

## 🧩 Key Features

### 🔄 MPC Integration

The MPC controller uses:

- PV forecast (e.g., Solcast)
- Load forecast
- Price forecast (EPEX/Nord Pool)
- Battery SOC  
- Cost minimization objective (`costfun = cost`)

The payload sensor builds the EMHASS API call and sends it every 15 minutes.

---

### 🔋 Battery Closed-Loop Control

This automation reads:

- `sensor.p_batt_forecast`  
- `sensor.soc_batt_forecast`  
- PV & load sensors  

and adjusts the inverter remote control accordingly every minute.

It supports:

- **Charging from the grid and PV**
- **No discharge during surplus periods**
- **Normal operation / discharge when beneficial**
- **SOC safety stop**

---

## 📊 Dashboard Example

For visualization, an **ApexCharts card** shows:

- Price forecast  
- PV forecast  
- Load forecast  
- Grid & battery power forecasts

Below is an example configuration snippet:

type: custom:apexcharts-card
...
# (chart YAML here)


You can add additional panels for day-ahead MPC outputs and SOC forecast.

🐞 Known Issues & Notes
⚠️ Solax Mode 8
There is a discrepancy in how “Enabled No Discharge” behaves in Mode 8 when PV is present:
According to documentation, Mode 8 submode should prevent discharge, supply house load from PV, and store surplus to battery.
In practice, the inverter may still draw from the battery or export to grid unexpectedly.
You can report this to SolaX support.

📌 Battery Safety
We enforce SOC bounds both in EMHASS config and within automations. For example, a safety stop is triggered when SOC < 22%.

🛠 Usage
Enable EMHASS server and configure Home Assistant to publish data
Schedule dayahead optimization at ~05:30
Run regular MPC optimizations every 15 minutes
Battery automation runs every minute to follow the MPC plan

📬 Contributions
If you have improvements, examples, or fixes, feel free to submit a Pull Request.

📜 License

This project is released under the MIT License.
