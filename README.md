## **General Information and Purpose**

**TE ID & Name:** TE-13 - MPC control of MFH energy system (Model Predictive Control for Multi-Family Homes)

**Description and purpose:** TE-13 provides **Model Predictive Control (MPC)** for **multi-family home (MFH) energy systems**, building on two complementary development tracks within META-BUILD.

From the **iDM** side, TE-13 extends the existing **iON energy optimiser**, originally developed for single-family homes, towards **multi-family home operation**. The controller aims to optimise **heat pump-based heating and DHW operation** over a forward horizon using **forecasts** (e.g. weather, price) and a **building/system model** to reduce operating cost and improve energy efficiency while maintaining comfort constraints. The MFH extension focuses on scaling to **multiple zones / dwellings**, requiring **zone-level monitoring and control** and MFH-specific operational constraints.

From the **EURAC** side, TE-13 is a **supervisory MPC** for MFH buildings that optimises the operation of **electrified energy systems**, including **heat pumps, thermal energy storage, DHW, PV and battery systems**. It operates in a **receding-horizon loop**: ingest measurements, update forecasts (weather, DHW, thermal load, PV), simulate with reduced-order models, compute an optimal plan via **dynamic programming (DP)** on discretised storage states, and dispatch the first action.

The common purpose of TE-13 is to **reduce energy use and operating cost**, **increase PV self-consumption**, **enable load shifting under price signals**, and **respect comfort and operational constraints**, with strong relevance for both pilot deployment and software-in-the-loop validation.

**Lead partner/Contact Information:**

- Lead (TE-13): **EURAC**
- Key contributors: **iDM**
- Supporting partner: **TUD**

**Target Front Runners/Pilots:**

- **FR1** - MFH pilot with **iDM heat pumps** and suitable MFH setup where zone-level monitoring/control is available or can be added, enabling multi-zone coordination and optimiser-driven scheduling
- **FR5** - Building with **heat pump(s)**, **DHW**, **phase-change-material (PCM) storage**, **electric energy storage** and **PV panels**

TE-13 is structured around supervisory predictive control of multi-asset building energy systems. Its architecture combines **measurements**, **forecasts**, **reduced-order models**, **optimisation logic** and **dispatch mechanisms**.

**iDM architecture flow:**

- Data acquisition from **heat pump system**, **zone module** (where deployed) and **meters**
- Forecast inputs including **weather**, optional **PV production estimates** and **price curves** where available
- Demand prediction using **building model** and **historical consumption**
- Optimisation engine (**iON**) computes the next-day optimal schedule / set-points
- Dispatch to **heat pump controller** and zone-level layer
- Monitoring and logging for continuous improvement

**EURAC architecture flow:**

- **BMS / EMS** and meters/sensors feeding an **edge controller** (IoT box)
- Local datastore for **time-series logging** and **model parameter persistence**
- Forecast services for **weather tuning**, **DHW**, **thermal load** and **PV power**
- Reduced-order model library for **heat pump**, **PCM**, **PV** and **BESS**
- **DP optimiser** performing shortest-path planning on a discretised state grid
- Dispatch of **supervisory set-points** (HP / PCM targets and BESS actions)

## **Functional Requirements**

Describe the core capabilities of the TE and the functions it provides.  
Focus on what the TE does, not how it is implemented.

- **24-Hour Predictive Optimisation:** Perform rolling-horizon optimisation of HVAC, DHW and storage systems, typically at **15-minute resolution (96 steps)**.
- **Comfort Constraint Enforcement:** Maintain indoor temperature and comfort indicators within user-defined bounds and support differentiated comfort profiles such as day/night or occupancy-aware settings.
- **Energy & Cost Minimisation:** Minimise total energy consumption and/or electricity cost while incorporating dynamic tariffs and price forecasts where available.
- **RES Self-Consumption Maximisation:** Optimise the interaction between **PV generation**, **storage** and **loads** to reduce grid import during high-price or peak periods.
- **System Constraint Handling:** Respect device limits such as power bounds, ramping, cycling, minimum on/off times and interactions across multiple subsystems.
- **Advisory & Automatic Operation:** Support **advisory mode** for pilots with limited actuation readiness and **automatic mode** where integration with control hardware is available.
- **KPI & Performance Reporting:** Produce outputs related to **energy savings**, **comfort compliance**, **RES utilisation** and **flexibility indicators** where applicable.
- **Edge-Deployable Supervisory Control:** Support receding-horizon supervisory control that can execute on constrained hardware and remain operational under intermittent connectivity.

## **Non-Functional Requirements**

- **Performance:**
  - Optimisation runtime target: **seconds to less than 1 minute per building**
  - Suitable for **daily and intra-day re-optimisation**
  - Not latency-critical for **sub-second control loops**
  - Full control cycle must complete **within the control interval on an edge device** (EURAC)
  - Designed for **constrained hardware** and deterministic runtime (EURAC)
- **Reliability and Availability:**
  - Designed for **continuous operation** during heating and cooling seasons
  - Supports **graceful fallback** to baseline control in case of missing forecasts or data
  - Resilient to **partial telemetry loss**
  - Supports **edge-first execution** to remain operational under intermittent connectivity (EURAC)
  - Supports **persistent states/parameters** for restart recovery and robust handling of missing data and infeasibility (EURAC)
- **Security (authentication, authorisation, data encryption, data privacy):**
  - **Authentication / secure access:** Requires secure access to heat pump, zone and BMS data and secure set-point writing
  - **Authorisation:** Access should be role-based, auditable and aligned with pilot IT policy
  - **Encryption:** Secure communication to BMS / EMS and APIs, typically via TLS-secured channels
  - **Privacy:** MFH operational data may be sensitive and should be governed with least-privilege access and local governance where edge execution is used
  - **Hardening:** Optional disk encryption and service hardening for edge deployment (EURAC)

## **Service Interfaces**

#### **Logical Interfaces**

- TE-13 exposes **supervisory optimisation interfaces** rather than low-level real-time control loops.
- Interfaces are exposed through **standardised APIs and data models defined in T4.1 (ED)** where applicable.

#### **Input Interfaces**

- **Building state** (temperatures, storage state)
- **Weather forecasts**
- **Load forecasts**
- **PV forecasts**
- **Tariff / price signals** (optional)

#### **Output Interfaces**

- **Optimal set-points and schedules**
- **KPIs and compliance indicators**
- **Advisory recommendations**
- **Supervisory dispatch actions** for controllable assets

#### **Interaction Modes**

- **Advisory mode** for pilots with limited actuation readiness
- **Automatic mode** where integrated with control hardware and pilot systems
- **Software-in-the-loop testing** during development and validation

#### **API Endpoints**

At the current stage, **API endpoints have not been fully specified yet**.

For this TE, the interface definition is currently described at **logical level**:

- **REST / integration APIs:** To be finalised during implementation and pilot integration
- **Primary integration mode:** Supervisory control integration through pilot BMS / EMS / controller interfaces
- **Data exchange:** Aligned with **TE-10 canonical models** where applicable
- **Output artefacts:** Schedules, set-points, KPIs, diagnostics and optimisation decisions

#### **UI Mockups (if applicable)**

- **Optional MPC dashboard:** Forecasts, PCM state-of-charge trajectory, last action, comfort constraint status and optimisation diagnostics
- **Supervisory control view:** Horizon schedule, active subsystem targets, PV/battery interaction and dispatch status
- **Performance view:** Energy savings, comfort compliance, RES utilisation and flexibility indicators

## **Data Model**

- **Main entities:**
  - **Building / Zone**
  - **HVACSystem**
  - **DHWTank**
  - **ThermalStorage / ElectricalStorage**
  - **Forecast**(weather, load, price, PV)
  - **ControlSchedule**
  - **KPIReport**
  - **OptimisationRun**

- **Relationships:**
  - One **Building** contains multiple **Zones** and controllable assets
  - One **Building** may include multiple subsystems such as HVAC, DHW, PV and storage
  - Each **optimisation run** produces one **control schedule** and one **KPI set**
  - **Forecasts** are linked to the optimisation horizon
  - **Schedules** and dispatch actions are linked to controllable assets and operating intervals

- **Logical schema / artefacts:**
  - buildings: {building_id, pilot_id, type, metadata}
  - zones: {zone_id, building_id, comfort_bounds, occupancy_profile}
  - hvac_systems: {hvac_id, building_id, type, nominal_power, operational_limits}
  - dhw_tanks: {tank_id, building_id, volume, temperature_bounds, state}
  - storage_assets: {storage_id, building_id, type, capacity, soc_bounds, charge_discharge_limits}
  - forecasts: {forecast_id, building_id, horizon_start, horizon_end, weather, load, pv, price}
  - optimisation_runs: {run_id, building_id, solver_type, created_at, runtime_ms, status}
  - control_schedules: {schedule_id, run_id, ts, target_asset, setpoint, action}
  - kpi_reports: {run_id, energy_saving, cost_saving, comfort_compliance, res_utilisation, flexibility_indicator}

## **Data Requirements**

When real pilot data are unavailable, **simulated or estimated profiles** may be used for validation.

| Data Category | Data Description | Source Type | Temporal Granularity | Spatial Scope | Historical Depth | Access Mode |
| --- | --- | --- | --- | --- | --- | --- |
| Indoor Temperature | Zone temperature | Sensors / BMS | 5-15 min | Zone / Building | As available | Read-only |
| Outdoor Conditions | Temperature, solar and related weather variables | Weather service | 15 min-hourly | Site / Region | Forecast horizon + recent history | Read-only |
| Energy Consumption | HVAC and DHW energy | Sub-meters / BMS | 15 min | Asset / Building | As available | Read-only |
| PV Generation | On-site RES production | Inverter / EMS | 15 min | Asset / Building | As available | Read-only |
| Tariffs / Prices | Electricity prices | Market / T4.4 / EPEX Spot / supplier API | Hourly / 15 min | Supplier / Region / Building | Forecast horizon + archive where available | Read-only |
| Control Set-points | HVAC / DHW / storage targets | MPC output | Event-based | Asset / Zone / Building | Not applicable | Read-write |

## **Integration and Dependencies**

- **External dependencies:** Weather forecast service, spot electricity price forecast service, cloud/server infrastructure, TE-11 for development and testing
- **System dependencies:** Commercial numerical solver, TE1 (brine-to-water heat pump), BMS / EMS interfaces for measurements and set-point writing, edge controller runtime and local datastore
- **Third-party integrations:** PV inverter interfaces and integration with pilot BMS (EURAC)
- **Edge/Cloud integration:** **Both** - **cloud-based execution** (iDM) and **edge-device execution** (EURAC)
- **Data space integration:** Inputs and outputs mapped to **TE-10 canonical model**; schedule and KPI JSON schemas exposed for dashboards and Front Runner reporting

**Integration points:**

- **TE-11:** Development and testing support through building models and calibration inputs
- **TE-10:** Canonical interoperability layer for input/output exchange where applicable
- **Pilot BMS / EMS:** Measurement ingestion, supervisory set-point writing and monitoring
- **PV / storage interfaces:** Integrated for self-consumption and flexibility use cases

## **Security and Privacy**

- **Data Sensitivity:** MFH energy, storage and zone-level time series can correlate with occupancy patterns and should be treated as **sensitive operational data**.
- **Access Control:** Role-based access to control functions and authenticated access to BMS interfaces.
- **Encryption:** Secure API and system communication, including **TLS-secured channels** where applicable.
- **Audit Logs:** Logging of control actions, optimisation decisions and error events for auditability and troubleshooting.
- **Privacy:** Access to high-resolution operational data should be restricted, auditable and aligned with pilot IT governance.

## **Current Status**

This section summarises the current maturity of TE-13, reflecting both the **iDM** and **EURAC** development tracks.

### **iDM**

**Achievements since D4.1:**

- MPC algorithms validated on multiple building archetypes
- Integration with **iDM control platforms** has progressed
- Support for **multi-asset optimisation** (HVAC + DHW + storage)
- Alignment with **flexibility objectives** and **demand response concepts**

**Validation:**

- Offline and semi-online testing using pilot data
- Cross-validation with simulation models (**SiL**)
- Pilot-specific calibration is ongoing

**Limitations:**

- Pilot heterogeneity in terms of **data quality** and **actuation readiness**
- Some pilots operate in **advisory mode only**
- Full grid-interactive operation depends on **T4.4 maturity**

### **EURAC**

**Progress since D4.1:**

- EURAC Research has extended its suite of **reduced-order models**
- Two additional modelling approaches have been introduced:
  - a **grey-box photovoltaic power forecasting model**
  - a **simplified electric energy storage model**
- The description of these models is presented in **D4.2**

## **Next Steps**

### **iDM**

1. **Tighter coupling with flexibility and grid services**
2. **Enhance user-aware comfort modelling**
3. **Integrate component models in SiL for MFH**, including:
   - hydraulics / heating circuit
   - heat dissipation (radiators and underfloor heating)
   - heat pump models
   - heat source model
4. **Calibrate control-oriented building models** from TE-11
5. **Formulate the Optimal Control Problem (OCP)**
6. **Implement the OCP**
7. **Validate the MPC strategy in the SiL testbench**
8. **Implement the MPC strategy in the Front Runner**

### **EURAC**

- The long-term goal toward **D4.3** is to develop a **simulation environment combining TRNSYS and Python** to model the system in **FR5** and test the MPC algorithm on it.
- The short-term next step is to **test and validate the thermal load model** presented in D4.1 on a building with a similar archetype to **FR5**.
