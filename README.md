# Torque Vectoring Control System — Formula Student 4WD EV

Final project (BSc, Mechanical Engineering), Ben-Gurion University of the Negev.
A closed-loop **Torque Vectoring (TV)** controller for the **BGRacing 2026 Formula Student**
four-wheel-drive electric race car, developed in **MATLAB/Simulink R2024b** and validated by
co-simulation with **IPG CarMaker 14.1.1**.

Each of the four in-wheel motors is controlled independently to generate a **direct yaw moment**
that corrects the car's cornering balance - more agile in tight corners, more stable at the
limit - without changing the driver's steering input.

---

## Highlights
- Linear **2-DOF bicycle model** used as the design plant and the real-time **yaw-rate reference**.
- **Gain-scheduled PI** yaw-moment controller (`pidtune`, phase margin 60°), output saturated to ±1000 Nm.
- **Load-based four-wheel torque allocation** (front/rear by load transfer, left/right for the yaw moment).
- Validated on the FSAE **Skid Pad** near the grip limit: **−3.33 % lap time**, **+7.5 % sustained lateral acceleration**, **−70 % RMS yaw-rate deviation** vs. an equal-split baseline.

---

## Repository structure
```
TV_Final_Project_V2024b/
├── TB_2024b.prj            # MATLAB/Simulink project — open this first
├── initial_param.m         # Parameters for the STANDALONE Simulink model
├── main_program.slx        # Standalone Simulink TV control model
├── resources/              # MATLAB project resources (auto-managed)
│
└── carmaker/               # IPG CarMaker co-simulation assets
    ├── initial_param.m     # Parameters for the CARMAKER run 
    ├── TV.mdl              # CM4SL model embedded in the CarMaker VehicleControl block
    ├── Nero_4EM.car        # Vehicle dataset (4 in-wheel motors, 11:1 driveline, aggressive driver)
    ├── FSUK_Sprint_2026.rd5 # Road / track
    └── TV_Testing_Skidpad  # Skid Pad TestRun (vehicle + road + maneuver + driver)
```
---

## Requirements
- **MATLAB / Simulink R2024b** + **Control System Toolbox** (`pidtune`)
- **IPG CarMaker 14.1.1** + **CarMaker for Simulink (CM4SL)**
- CarMaker Formula Student add-on / `Nero_4EM` base vehicle

---

## How to run

### A. Standalone Simulink
1. Open `TB_2024b.prj` in MATLAB R2024b.
2. Run `initial_param.m` (root).
3. Open and run `main_program.slx`.

### B. CarMaker co-simulation (the validation in the report)
1. Copy the CarMaker assets into your CarMaker project tree:
   - `carmaker/Nero_4EM.car`          → `<CMProject>/Data/Vehicle/`
   - `carmaker/FSUK_Sprint_2026.rd5`   → `<CMProject>/Data/Road/`
   - `carmaker/TV_Testing_Skidpad`    → `<CMProject>/Data/TestRun/`
2. Run `carmaker/initial_param.m` in MATLAB, then start CarMaker for Simulink so `carmaker/TV.mdl`
   is embedded in the **VehicleControl** block.
3. Open the `TV_Testing_Skidpad` TestRun and run it.
4. For the baseline, set the corrective moment to zero (`M_TV = 0`); compare against the TV run and
   plot the logged yaw rate, lateral acceleration, and wheel torques.

> **Powertrain note:** the 4WD reduction (**11:1**) lives in the CarMaker driveline (per-corner
> gearbox), and the motors are driven in external torque-request mode. The reference yaw rate is
> clamped to a maximum admissible lateral acceleration.

---

## Results (Skid Pad, aggressive driver, μ = 1.0)
| Metric | Baseline | TV | Change |
|---|---|---|---|
| Averaged lap time [s] | 6.945 | 6.713 | −0.232 (−3.33 %) |
| Mean lateral acceleration [m/s²] | 7.23 | 7.76 | +7.5 % |
| RMS yaw-rate deviation [rad/s] | 0.0195 | 0.0058 | −70 % |

Open limitations / next steps: ±1000 Nm actuator saturation and the absence of an anti-windup loop;
a back-calculation anti-windup scheme and a re-validated velocity-scheduled gain set are planned.

---

## Authors & supervision
- **Itai Groisman** · **Tomer Tzahor**
- Advisor: **Guy Zaidner**, Mechanical Engineering Department, Ben-Gurion University of the Negev
- Project **26-37**, BGRacing Formula Student.
