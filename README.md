# Torque Vectoring Control System — Formula Student 4WD EV

Final project (BSc, Mechanical Engineering), Ben-Gurion University of the Negev.
A closed-loop **Torque Vectoring (TV)** controller for the **BGRacing 2026 Formula Student** four-wheel-drive electric race car, developed in MATLAB/Simulink (R2024b) and validated by co-simulation with **IPG CarMaker**.

By controlling the torque of each of the four in-wheel motors independently, the system generates a **direct yaw moment** that corrects the car's cornering balance — making it more agile in tight corners and more stable at the limit — without changing the driver's steering input.

---

## Highlights

- Linear **2-DOF bicycle model** used both as the design plant and as a real-time **yaw-rate reference generator**.
- **Gain-scheduled PI** yaw-moment controller (tuned with `pidtune`, ωc = 40 rad/s, PM = 60°).
- **Load-based four-wheel torque allocation** that biases torque front/rear with the instantaneous longitudinal load transfer and left/right to realize the commanded yaw moment.
- Validated on the FSAE **Skid Pad** near the grip limit: **−3.33 % lap time**, **+7.5 % sustained lateral acceleration**, **−70 % RMS yaw-rate deviation** vs. an equal-split baseline.

---

## Repository structure

```
TV_Final_Project_V2024b/
├── TB_2024b.prj          # MATLAB/Simulink project file — open this first
├── initial_param.m       # Loads all vehicle, powertrain & controller parameters into the workspace
├── main_program.slx      # Complete Simulink TV control model (R2024b)
├── resources/project/    # MATLAB project resources
├── .gitattributes
└── .gitignore
```

> The Simulink model is parameterized: `initial_param.m` is the single source of truth for all constants (geometry, mass, cornering stiffness, gear ratio, motor limits, gain schedule, filter coefficients).

---

## Requirements

- **MATLAB / Simulink R2024b**
- **Control System Toolbox** (for `pidtune`)
- **IPG CarMaker 14.1.1** + **CarMaker for Simulink (CM4SL)** — for the co-simulation validation
- (Optional) the CarMaker Formula Student add-on / `Nero_4EM` vehicle dataset

---

## How to run

### 1. Standalone Simulink
1. Open `TB_2024b.prj` in MATLAB R2024b (sets the path automatically).
2. Run `initial_param.m` to load all parameters into the workspace.
3. Open and run `main_program.slx`.

### 2. CarMaker co-simulation
1. Launch IPG CarMaker and open the project (see `resources/project/` / the CarMaker project files).
2. Start CarMaker for Simulink so `main_program.slx` is embedded in the **VehicleControl** block.
3. Run `initial_param.m`, then start the desired TestRun (e.g. Skid Pad) from the CarMaker GUI.
4. Post-process / plot the logged channels (yaw rate, lateral acceleration, wheel torques) to reproduce the report figures.

> **Powertrain note:** the 4WD reduction (11:1) must be present in the CarMaker driveline (per-corner gearbox), and the motors must be driven in external torque-request mode. The reference yaw rate is built from the steering angle and the understeer gradient and is clamped to a maximum admissible lateral acceleration.

---

## Controller overview

```
sensors ─► state/velocity estimation ─► yaw-rate reference (bicycle model)
                                              │
                  measured yaw rate ──►(−)──► PI (gain-scheduled) ──► M_TV (±1000 Nm)
                                                                        │
                          load-based front/rear split (λ) + L/R differential
                                                                        │
                                              four wheel torque commands (saturated)
```

- **Reference:** `ψ_des = v_x · δ / (l + K_us · v_x²)`, saturated by `a_y,max`.
- **Control law:** velocity-scheduled PI on the yaw-rate error, output saturated to ±1000 Nm.
- **Allocation:** `λ = b/l − (h/(g·l))·a_x` sets the front share; per-axle differential `Δτ = (r/(G_r·t))·M_z`; the four commands sum to the driver torque while delivering the target moment.

---

## Results (Skid Pad, aggressive driver, μ = 1.0)

| Metric | Baseline | TV | Change |
|---|---|---|---|
| Averaged lap time [s] | 6.945 | 6.713 | −0.232 (−3.33 %) |
| Mean lateral accel. [m/s²] | 7.23 | 7.76 | +7.5 % |
| RMS yaw-rate deviation [rad/s] | 0.0195 | 0.0058 | −70 % |

Known limitations / next steps: the corrective moment saturates at ±1000 Nm authority, and there is no anti-windup yet — a back-calculation anti-windup loop and a re-validated velocity-scheduled gain set are the planned improvements.

---

## Authors & supervision

- **Itai Groisman**
- **Tomer Tzahor**
- Advisor: **Guy Zaidner** — Mechanical Engineering Department, Ben-Gurion University of the Negev
- Project **26-37**, BGRacing Formula Student.

## Citation / report

The full report and extended summary accompany this code. Please cite the project report when using this work.
