# Solax-EMHASS
![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Compatible-blue)
![EMHASS](https://img.shields.io/badge/EMHASS-Integrated-green)
![License](https://img.shields.io/github/license/ralfna/Solax-Emhass)
![Last Commit](https://img.shields.io/github/last-commit/ralfna/Solax-Emhass)

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


- `sensors.yaml`: Template sensors for forecasts and MPC inputs  
- `rest_commands.yml`: REST definitions for EMHASS API calls  
- `*.yml` automations: MPC control & battery automations

---

## 🧩 Key Features

### 🔄 MPC Integration
- 15-minute MPC horizon using EMHASS
- Cost-optimized scheduling based on:
  - PV forecast
  - Load forecast
  - Day-ahead electricity prices
  - Battery SOC constraints
- Automatic REST-based triggering from Home Assistant

The MPC controller uses:

- PV forecast (e.g., Solcast)
- Load forecast
- Price forecast (EPEX/Nord Pool)
- Battery SOC  
- Cost minimization objective (`costfun = cost`)

The payload sensor builds the EMHASS API call and sends it every 15 minutes.


### 🧠 Dictionary-Based Sensor Design
All forecast and MPC output sensors are stored in **dictionary (dict) format** instead of flat time-series sensors.

This design choice significantly simplifies:
- Start-time alignment
- Time-index calculations
- MPC horizon handling
- Synchronization between PV, load, grid and battery forecasts

Each dict entry contains explicit timestamps, allowing robust time arithmetic without relying on implicit sensor update intervals.

### 🔋 Battery Closed-Loop Control

- Minute-based closed-loop execution
- Follows MPC scheduled battery power (`p_batt_forecast`)
- Supports:
  - Grid charging
  - PV charging
  - No-discharge periods
  - Normal operation fallback
- Additional SOC safety stop outside EMHASS

This automation reads:

- `sensor.p_batt_forecast`  
- `sensor.soc_batt_forecast`  
- PV & load sensors  

and adjusts the inverter remote control accordingly every minute.
---

📌 Battery Safety
We enforce SOC bounds both in EMHASS config and within automations. For example, a safety stop is triggered when SOC < 22%.

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


## 🛠 EMHASS Add-on Compatibility Notes

The Home Assistant EMHASS **Add-on installation is not fully compatible with the file path structure used in the EMHASS standalone setup**.

Several paths (e.g. for forecast CSV files) differ from the upstream documentation and cannot be used reliably inside the add-on environment.

### Workaround Used in This Project

To avoid filesystem path issues, this project:

1. Executes EMHASS `predict` via REST
2. Publishes the **load forecast as a Home Assistant sensor**
3. Converts the forecast into **dictionary format**
4. Feeds the resulting dict sensor back into subsequent MPC and battery control logic

This approach:
- Avoids direct file access inside the EMHASS container
- Keeps all data exchange within Home Assistant
- Improves robustness and portability across installations

While this deviates from the standalone EMHASS workflow, it has proven to be reliable in long-term operation with the Home Assistant add-on.


🛠 Usage
Enable EMHASS server and configure Home Assistant to publish data
Schedule dayahead optimization at ~05:30
Run regular MPC optimizations every 15 minutes
Battery automation runs every minute to follow the MPC plan

📬 Contributions
If you have improvements, examples, or fixes, feel free to submit a Pull Request.

## 🔌 SolaX Modbus Integration

This project relies on the excellent Home Assistant SolaX Modbus integration:

👉 https://github.com/wills106/homeassistant-solax-modbus

It provides detailed inverter and battery metrics and allows reliable remote control of operating modes required for MPC-based optimization.



📜 License

This project is released under the MIT License.
