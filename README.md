# Optimization of Hybrid Solar-Wind Energy System for Hospital in London

[![MATLAB](https://img.shields.io/badge/MATLAB-R2023b-orange)](https://www.mathworks.com/products/matlab.html)

This repository contains a MATLAB-based optimization project focused on designing a hybrid energy system for a hospital. The system integrates solar photovoltaics (PV), wind turbines, and battery storage, aimed at reducing energy costs and minimizing dependency on the grid while ensuring reliable energy supply for critical medical operations.

## Introduction

Hospitals are among the most energy-intensive facilities, requiring continuous and reliable power sources to support medical operations. This project develops an energy optimization model to design a hybrid solar-wind energy system for a hospital located in London, UK. By incorporating solar PV, wind turbines, and battery storage, the goal is to provide a self-sufficient energy system that minimizes grid dependency, reduces operational costs, and supports sustainability goals.

The hybrid system is optimized to meet the hospital's energy demands while maximizing the utilization of renewable energy resources, considering the hospital's daily load profile, environmental conditions, and space constraints.

## Project Description

This system is optimized using a Genetic Algorithm (GA) to determine the optimal configuration that minimizes the total system investment cost over a 10-year lifetime, while fulfilling the following constraints:

- **SOC Constraints:** The state of charge (SOC) of the battery must remain within 50% to 90%.
- **Power Supply Constraints:** The system must meet the energy load requirements, with any excess energy being exported.
- **Space Constraints:** The total space required for the PV panels and wind turbines must not exceed the usable area of the hospital’s rooftop or nearby land.

The optimization procedure also integrates a financial analysis to evaluate the feasibility and payback period of the system based on energy savings and export revenues via the Smart Export Guarantee (SEG) scheme.

## Project Methodology
- Real-world data integration (irradiance, temperature, wind, and load)
- Physical modeling of solar, wind, and battery systems
- Control system design taking into consideration the battery limit and excess power exportation
- Cost and economic analysis (CAPEX + OPEX)
- Constrained multi-variable optimization
- Visual analytics of system behavior and energy flows

## System Specifications

### 1. Solar PV System
- **Rated Power:** 485W per panel
- **Efficiency:** 22.4%
- **Temperature Coefficient:** -0.29% per °C
- **Degradation Rate:** 0.4% per year

### 2. Wind Turbines
- **Rated Power:** 2kW per turbine
- **Efficiency:** 92%
- **Wind Speed:** Optimized for London’s average wind speeds (4-6 m/s)

### 3. Battery Storage
- **Capacity:** 4560 Wh per cell
- **Efficiency:** 98%
- **Depth of Discharge (DoD):** 95%

## Optimization Methodology

## 🧮 Model Inputs

**Time Resolution:**

$$
\Delta T = 0.5 \text{ hr} \quad \text{(12 days × 24 hr = 288 hours)}
$$

**Scaled Load Profile:**

$$
P_{\text{load}}(t) = P_{\text{load,raw}}(t) \cdot (1 + g)^{y}, \quad g = 0.025, \quad y = 10
$$

---

## System Equations

### PV Output

$$
P_{\text{PV}}(t) = N_{\text{pv}} \cdot \eta_{\text{PV}} \cdot \frac{P_{\text{STC}}}{G_{\text{STC}}} \cdot \left(1 + C_T(T(t) - T_{\text{STC}})\right) \cdot G(t) \cdot (1 - r_{\text{deg}})^{y-1}
$$

###  Wind Turbine Output

$$
P_{\text{wind}}(t) = N_{\text{turb}} \cdot \left( A + B \cdot v(t) + C \cdot v(t)^2 \right) \cdot P_{\text{turb}}
$$

Where \( A, B, C \) are constants computed from cut-in \(v_{ci} = 2.5\) and rated speed \(v_r = 10\).

### Battery State of Charge (SOC)

$$
SOC(t+1) = SOC(t) + \frac{\eta_{\text{bat}} \cdot P_{\text{bat}}(t) \cdot \Delta T}{N_{\text{cell}} \cdot C_{\text{cell}}}
$$

Battery charge/discharge power:

- **If surplus:**

$$
P_{\text{bat}}(t) = -\eta_{\text{DCDC}} \cdot \left(P_{\text{total}} - \frac{P_{\text{load}}(t)}{\eta_{\text{ACDC}}} \right)
$$

- **If deficit:**

$$
P_{\text{bat}}(t) = -\frac{1}{\eta_{\text{DCDC}}} \cdot \left(P_{\text{total}} - \frac{P_{\text{load}}(t)}{\eta_{\text{ACDC}}} \right)
$$

With:

$$
P_{\text{total}} = \eta_{\text{DCDC}} \cdot P_{\text{PV}}(t) + \eta_{\text{ACDCw}} \cdot P_{\text{wind}}(t)
$$

---

## 🎯 Objective Function

Minimize total cost over 10 years:

$$
C(X) = N_{\text{pv}} (C_{\text{inv,panel}} + OPEX_{\text{PV}}) + N_{\text{cell}} (C_{\text{inv,cell}} + OPEX_{\text{Battery}}) + N_{\text{turb}} (C_{\text{inv,turb}} + OPEX_{\text{Wind}})
$$

---

## 📌 Constraints

### Inequality Constraints

$$
\begin{aligned}
& SOC_{\text{min}} \leq SOC(t) \leq SOC_{\text{max}} \\
& \max(P_{\text{unsup}}(t)) = 0 \\
& SOC(0) = SOC(T_{\text{end}}) \\
& N_{\text{pv}} \cdot A_{\text{pv}} + N_{\text{turb}} \cdot A_{\text{turb}} \leq A_{\text{usable}}
\end{aligned}
$$

### Variable Bounds

$$
\begin{aligned}
1 &\leq N_{\text{pv}} \leq \left\lfloor \frac{A_{\text{usable}}}{A_{\text{pv}}} \right\rfloor \\
10 &\leq N_{\text{cell}} \leq 1000 \\
0 &\leq N_{\text{turb}} \leq \left\lfloor \frac{A_{\text{usable}}}{A_{\text{turb}}} \right\rfloor \\
SOC_{\text{min}} &\leq SOC_{\text{init}} \leq SOC_{\text{max}}
\end{aligned}
$$

---

## 🧬 Optimization Method

MATLAB’s **Genetic Algorithm** is used to solve:

$$
\min_{X \in \mathbb{R}^4} \quad C(X) \quad \text{subject to constraints}
$$

**GA Settings:**

- Population Size: 500  
- Max Generations: 10,000  
- Function Tolerance: \(10^{-12}\)  
- Constraint Tolerance: \(10^{-3}\)

The **genetic algorithm** (`ga()`) used to find the optimal configuration is as seen below:

```matlab
[Xopt, cost] = ga(obj, 4, [], [], [], [], Xl, Xu, nlconstraints, [1 2 3], options)


The optimization model is implemented in MATLAB and follows these steps:
1. **Input Data:** Load the hospital's energy load profile, solar irradiance, wind speed, and temperature data.
2. **System Sizing:** Calculate the power generated by each component (PV, wind, and battery).
3. **Genetic Algorithm:**
   - Optimize the number of PV panels, wind turbines, and battery cells.
   - Minimize the total system investment cost, including both capital and operational expenses.
   - Constraints include meeting energy load requirements, staying within battery SOC limits, and respecting available land area for installation.
4. **Results:** The model outputs the best configuration of the system, along with financial metrics like payback period and internal rate of return (IRR).

## Steps to Run the Project

### Prerequisites:
- MATLAB (R2020b or later)
- MATLAB Optimization Toolbox (for Genetic Algorithm)
- Data files (dailydata12monthsaverage.csv, wind_speed.txt, load3.txt, etc.)

### Steps:
1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/hybrid-energy-system.git
