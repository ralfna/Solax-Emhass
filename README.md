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

