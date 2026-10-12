# Microduck robotics learning roadmap

Created 2026-10-12. Personal learning plan: 2–3 hours per week for six months, with hardware expected in 4–6 months. This is a proposed curriculum, not a report of experiments already completed.

## Direction

Use existing software engineering strengths to develop robotics judgment: define a physical task, identify what the robot can observe and control, measure behavior, isolate failures, and justify a design under uncertainty.

The initial career direction is robotics systems, simulation/evaluation, and autonomy integration. This is a recommendation based on your background, not a hiring forecast. Controls research, perception research, and embedded motor control are possible later specializations with different foundations.

Conceptual understanding still needs a quantitative core. Learn enough vector/matrix math, basic calculus, probability, and feedback control to check an agent's implementation. Syntax can be delegated; units, frames, timing, assumptions, and interpretation cannot.

The main project is one evolving Microduck test bench. Keep the existing VelStand policy as the baseline. Expand the bench from manual experiments into reproducible evaluations and then runtime integration. Avoid simultaneously starting a new gait, SLAM stack, and custom hardware project.

## Weekly rhythm and scope

- 30–40 minutes: one concept, selected reading or video, and a prediction.
- 75–110 minutes: one focused experiment, including setup and debugging.
- 15–20 minutes: plots or video inspection and a short experiment note.

Use 24 core weeks plus two buffer weeks. If setup consumes a session, it counts. If a checkpoint is not met, extend that month and drop a stretch goal. Do not compensate by silently increasing the weekly time commitment.

Every note answers: What did I predict? What was changed? What was held constant? What did the data and video show? What remains uncertain? What decision follows?

## Month 1 — Describe the robot and establish a baseline

Concepts: joint configuration versus body pose; radians and angular velocity; body versus world frames; rotation representations; forward kinematics; observations, actions, and commands; training versus inference; physics rate versus policy rate.

Learn vectors, sine/cosine, and rotation matrices through specific examples. Start with planar rotation before studying quaternions. Quaternions represent orientation; they are not four independent angles.

| Week | Activity |
| --- | --- |
| 1 | Trace one forward command from terminal input to the policy and motor targets. Sketch the loop. Observe idle, slow forward walking, and turning. |
| 2 | Draw body/world/head/foot frames. Rotate a forward vector by 90 degrees and predict its world direction. Identify joint axes in the actual model. |
| 3 | Record short baseline trials: idle, forward, backward, and turning. Restart from a consistent state and repeat each three times. |
| 4 | Summarize commanded versus achieved velocity, drift, tilt, falls, and visible behavior. Define a provisional fall criterion using orientation/contact, not height alone. |

Deliverable: a one-page system sketch and a baseline report with two plots and representative videos. Three repeats are descriptive, not proof of reliability.

Checkpoint: explain why asking for 0.3 m/s does not guarantee achieving 0.3 m/s, and why zero command still produces motor activity. Explain how a body-frame forward command changes direction after a turn.

Repo entry points: `Launch Walking.command`, `scripts/infer_policy.py`, `src/mjlab_microduck/robot/microduck_constants.py`, and the model XML. The policy's 14 action joints and the runtime's motor list are separate contracts; verify the mouth joint and ordering rather than assuming identical arrays.

## Month 2 — Understand feedback and actuator limits

Concepts: feedback error; proportional and derivative control; damping; overshoot; settling time; torque/current/voltage limits; friction; backlash; delay; contact and balance. Holding a joint target is different from maintaining whole-body balance.

| Week | Activity |
| --- | --- |
| 5 | Use an isolated simulated joint or simple pendulum to study a position step. Predict what increasing proportional gain will change. Do not modify the walking policy to do this. |
| 6 | Compare two or three gain/damping settings. Plot response and measure overshoot and settling time. If the BAM parameter interface prevents a clean comparison, use a separate toy PD example. |
| 7 | Run the same Microduck command sequence with baseline, one-step, and two-step action delay. At 50 Hz these are nominally 0, 20, and 40 ms of added action delay. |
| 8 | Vary one floor-friction condition in the same model. Compare response, drift, and falls. Write down which explanation the experiment supports and which it cannot distinguish. |

Deliverable: a small control-response report and one repeatable delay experiment added to the test bench.

Checkpoint: explain why increasing update frequency, gain, or smoothing is not automatically an improvement. Explain why a trained policy can fail when actuator behavior changes.

Repo entry points: `scripts/infer_policy.py` has `--delay` and `--foot-friction`; the BAM model is in `src/mjlab_microduck/actuator/friction_dr_bam.py`. `scripts/testbench_sim2real.py` illustrates joint response measurement but requires its own testbench policy; it is not an automatic drop-in experiment for VelStand. Keep the normal walking baseline on BAM. Changing to `--no-bam` changes the actuator assumptions.

## Month 3 — Learn what sensors can actually tell you

Concepts: sensor bias versus noise; timestamps; filtering lag; estimates versus simulator truth; uncertainty; observability; heading and odometry drift. Learn mean/variance and the basic idea of covariance through plots.

| Week | Activity |
| --- | --- |
| 9 | Make a sensor inventory: encoders, gyro, accelerometer/projected gravity, and any derived quantities. Label what is directly measured, estimated, or simulation-only. |
| 10 | Build a toy heading estimate by integrating a gyro trace. Add a small synthetic bias to a copy of the trace and compare against simulator heading. Handle angle wraparound. |
| 11 | Compare raw and lightly filtered traces. Measure noise reduction and added lag during a turn. Keep this offline; do not alter the policy observation pipeline. |
| 12 | Explain an estimation failure from the plots. Optionally compare a simple roll/pitch complementary filter against truth; this does not solve yaw drift. |

Deliverable: a notebook or script showing heading drift and the filtering tradeoff, plus the sensor inventory.

Checkpoint: explain why integrating an IMU does not give indefinitely accurate position, and why gravity can constrain roll/pitch but cannot supply absolute yaw. Separate truth used for evaluation from signals allowed in a deployable controller.

Do not inject new sensor channels into the trained 61D policy input. Any experimental estimator stays beside the policy until its contract is deliberately changed and validated.

## Month 4 — Exercise the actual runtime and failure handling

Concepts: periodic loops; deadlines and jitter; simulation time versus wall time; IPC; command ownership; state machines; stale commands; watchdogs; startup/reconnect behavior; logs and replay.

The official `microduck` repo documents `scripts/duck-sim`: real robot daemons connected to a simulated MuJoCo body. Hardware drivers remain outside this simulation. The plain-process mode is the relevant starting point on the Mac; the systemd container mode is a Linux exercise.

| Week | Activity |
| --- | --- |
| 13 | Set up the runtime repo and plain `duck-sim` mode, following its current docs. Budget this whole session for installation. Record both repository revisions. |
| 14 | Repeat one baseline experiment through `robotctl`. Trace the command, runtime state, sensor input, policy output, and body response. Compare with the Python inference path. |
| 15 | In simulation, interrupt the command source and then the body connection separately. Record what the existing runtime actually does; do not assume a timeout or stop mechanism exists. |
| 16 | Specify desired stale-command behavior and recovery conditions. If time permits, add a tiny external command supervisor using supported APIs; otherwise deliver the fault report and design. |

Deliverable: an architecture sketch and a fault table: trigger, expected behavior, observed behavior, evidence, unresolved gap. Command-zeroing requests idle; it is not equivalent to holding targets, disabling torque, or an emergency stop.

Checkpoint: explain what should happen when a process dies while a walking command is active. Distinguish command-source failure from a lost sensor/motor connection.

ROS 2 is a small literacy exercise here, not a rewrite. Learn nodes, topics, services/actions, transforms, QoS, and recording/replay from the official beginner tutorials. Run one small publish/subscribe example if time allows. Do not force Microduck's existing Rust/IPC architecture into ROS. Use a suitable Linux environment if native Mac setup becomes a distraction; defer setup rather than consuming the month.

## Month 5 — Understand learned control and test sim-to-real assumptions

Concepts: state versus observation; action; reward; episode termination; discounted return; actor versus critic; PPO at a conceptual level; observation normalization; reward exploitation; curricula; domain randomization; system identification.

| Week | Activity |
| --- | --- |
| 17 | Map the walking/VelStand task to observation, action, reward, reset, and termination. Explain why runtime inference needs the actor but not the training critic. |
| 18 | Inspect three reward terms and construct a behavior that could exploit a poorly chosen sign or gate. Evaluate a few fixed simulated states rather than immediately launching training. |
| 19 | Run a small robustness battery against the unchanged baseline: nominal, extra action delay, and reduced friction, five repeated trials each. Record seeds or initial conditions. |
| 20 | Report failures alongside successful-trial tracking. Audit the observation order, normalization/export path, timing, actuator model, and command ranges. Write a sim-to-real assumption list. |

Deliverable: a robustness report and a policy deployment-contract checklist. A sweep tests robustness within a model; it does not establish real-world reliability or prove the chosen randomization ranges match hardware.

Checkpoint: explain why total training reward can increase while the desired behavior gets worse, and why broad randomization can sacrifice nominal performance. Distinguish fitting the simulator to measurements from training robustness to uncertain parameters.

Training is optional, not a graduation requirement. This repo's parallel MuJoCo Warp training needs a suitable NVIDIA/CUDA environment; the Mac CPU viewer is a separate path. If choosing a training experiment, use the supported remote GPU path, set an explicit budget, and run the mandatory 64-environment/5-iteration smoke test before a longer run. Preserve the shared observation contract, BAM startup/friction behavior, reward signs, and baked normalizer export. Do not hand-convert checkpoints.

Read `docs/velstand_policy.md` as an engineering case study. Its recorded lineage is useful for hypothesis formation; verify the local ONNX identity before attributing a documented version's results to your file.

## Month 6 — Ship a small, defensible robotics project

Core capstone: a reproducible Microduck behavior evaluation tool using the real runtime in simulation. Given a fixed command sequence, it runs trials, records telemetry, produces a report, and documents how commands expire or recover after interruption.

| Week | Activity |
| --- | --- |
| 21 | Freeze a small requirement: idle → walk a short sequence → turn → idle. Choose limits and success/failure criteria before running the final evaluation. |
| 22 | Automate the sequence and reporting. Reuse months 1–5 scripts. Keep reports plain if building a UI would displace robotics work. |
| 23 | Run nominal and two degraded conditions. Include failed trials and a video that challenges the metrics. Test that stop behavior matches the chosen semantics. |
| 24 | Package a demo, reproduction instructions, two useful plots, limitations, and one decision memo. Record code revisions, ONNX hash, scene/actuator settings, commands, seeds, and time basis. |

Deliverable: a short demo plus a report someone else can reproduce. The decision memo answers: What did I choose? What alternatives did I consider? What evidence changed my view? What would make me reverse the decision?

Stretch only: an external supervisor that stops forward commands when simulated depth reports an obstacle. This adds basic perception without retraining locomotion. Define invalid/stale depth behavior and test it. It is an obstacle-stop demo, not collision-free navigation or a safety guarantee. Alternatively, expose read-only telemetry through a small ROS 2 bridge; choose one stretch, not both.

Weeks 25–26: catch up, improve reproducibility, or perform manufacturer-guided hardware onboarding if the robot has arrived. Hardware is not required to finish this plan.

## When the physical robot arrives

Follow current manufacturer setup and calibration instructions. Establish the supported stop/power-off procedure before experiments. Begin with supplied policies and short, low-speed trials in the recommended setup. Log the same signals as in sim, noting anything the hardware cannot observe directly.

Compare one controlled condition at a time: tracking, response delay, drift, recovery, voltage, and actuator limits where available. Record a mismatch as a finding instead of immediately tuning it away. Rehearse any altered policy in simulation with the matching observation and actuator contract before deployment.

Simulation does not validate the physical bus, mounting/calibration, temperatures, actual surface friction, structural wear, battery behavior, or deployment safety. Keep the sim results explicitly scoped.

## Learn with agents while keeping ownership

Before asking for code, state the hypothesis, units, coordinate frames, timestamps, inputs, allowed changes, and success criteria. Ask the agent to identify hidden simulator truth, sign conventions, and mismatched training/runtime contracts.

Use agents to write harnesses, inspect call paths, plot traces, and suggest failure hypotheses. Personally check at least one numerical example, inspect the plots and videos, and explain the result without the agent. An implementation you cannot assess is unfinished learning.

Use this experiment template:

```text
Question / prediction:
Independent variable:
Baseline and held-constant settings:
Sensors available to the controller:
Simulator truth available only to evaluation:
Metrics / failure definition:
Trial count and seeds / initial conditions:
Observed result and video evidence:
Limitations / next decision:
```

## Selected resources, not a course backlog

- [Modern Robotics, Lynch and Park](https://modernrobotics.northwestern.edu/): selected Chapter 2–4 material for configurations, frames, rotations, and kinematics; Chapter 11 for feedback control. Work through a few numerical examples instead of trying to finish the book.
- [MIT Underactuated Robotics](https://underactuated.mit.edu/): selected pendulum, simple walking, and stochastic-system sections. Use intuition first; postpone advanced proofs and trajectory optimization.
- [Official Microduck runtime simulation guide](https://github.com/pollen-robotics/microduck/blob/main/docs/robot/simulation.md): month 4 setup and the boundary between simulation and actual drivers.
- [Official ROS 2 tutorial source](https://github.com/ros2/ros2_documentation/blob/jazzy/source/Tutorials.rst) and [record/replay tutorial](https://github.com/ros2/ros2_documentation/blob/jazzy/source/Tutorials/Beginner-CLI-Tools/Recording-And-Playing-Back-Data/Recording-And-Playing-Back-Data.rst): a small introduction to robotics middleware; not a prerequisite for the early Microduck work.
- [Sim-to-Real: Learning Agile Locomotion for Quadruped Robots](https://arxiv.org/abs/1804.10332): month 5 reading about actuator modeling and robustness/nominal-performance tradeoffs. Quadruped findings inform questions; they do not establish Microduck behavior.

## After six months

Choose a direction using the work you enjoyed and the gaps your experiments exposed. Systems/evaluation: extend runtime tooling, fault injection, and repeatable hardware comparisons. Autonomy/perception: add geometry, sensor fusion, camera/depth processing, and a simple navigation project. Controls/locomotion: invest more deeply in dynamics, optimization, and measured policy experiments.

In months 7–12, practice Linux debugging and enough C++ to understand a robotics component's timing, memory, and interfaces. Make one useful upstream contribution. Review actual job descriptions for your preferred role/location and revise the skill plan rather than treating robotics as one uniform job.

Six months at this pace builds foundations and a credible first project; it does not by itself establish job readiness. The longer 1–3-year goal leaves time for hardware experience, deeper specialization, and work on a second robot/platform.
