# Why I am learning robotics

Written from my goals shared on 2026-10-12. The practical curriculum is in
[the six-month study roadmap](learning-roadmap.md).

## Background and motivation

I am a self-taught software engineer with experience in web, mobile, and backend
services. I currently work in payments, where reliability and the consequences
of failure matter.

I expect AI agents to take on an increasing share of implementation. I want to
develop the judgment that still determines what to build: product decisions,
tradeoffs, direction, taste, and opinions grounded in evidence. Product
engineering feels like a natural direction for my existing skills.

I also want to explore robotics as a frontier for useful AI in the physical
world. Generating software is one part of building robots; physical behavior,
sensing, control, reliability, and deployment introduce other hard problems.

My longer-term goal is to be able to apply for robotics software engineering
roles in roughly 1–3 years. There is no fixed deadline. The next six months are
for building foundations and learning which parts of the field I want to pursue.

## Constraints and learning preferences

- Available time: 2–3 hours per week.
- My physical Microduck is expected in 4–6 months; the initial plan is entirely
  simulation-based and can be completed without the robot.
- Focus on concepts, system behavior, experiments, and engineering decisions.
  Python syntax and library trivia are secondary.
- Use agents to help implement tools, while personally understanding the units,
  frames, assumptions, evidence, and conclusions.
- Learn enough math and implementation detail to inspect and debug the work.
  Conceptual learning should still include concrete numerical examples.

## How Microduck supports the goal

This fork contains the simulation and RL training environment from
[`pollen-robotics/microduck_rl`](https://github.com/pollen-robotics/microduck_rl).
The separate [`pollen-robotics/microduck`](https://github.com/pollen-robotics/microduck)
repository contains the robot runtime and a mode that runs its daemons against
a simulated body. See its [simulation guide](https://github.com/pollen-robotics/microduck/blob/main/docs/robot/simulation.md)
when reaching month 4.

The main learning project is a Microduck behavior test bench: repeatable command
sequences, telemetry, failure scenarios, plots, and a report explaining results
and limitations. Keep a supplied policy as the baseline before exploring training.

Initial areas to explore are robotics systems, simulation/evaluation, and
autonomy integration. This is a learning direction, not a settled specialization
or a claim of current job readiness.

Progress means being able to explain a physical failure, design a useful
experiment, reproduce a result, and justify a tradeoff. Watching a convincing
demo or getting an agent to generate code does not establish those abilities.

For future assistants: read this context, the roadmap, and `AGENTS.md` before
proposing work. Keep exercises within the weekly budget, distinguish simulated
evidence from hardware validation, and do not mark study milestones complete
without evidence. Follow the repository's sim-to-real invariants.

## Continue on another laptop

Clone the learning branch, then install dependencies using the repository's
standard `uv` setup:

```bash
git clone --branch develop https://github.com/jeffgat/microduck_rl.git
cd microduck_rl
git remote add upstream https://github.com/pollen-robotics/microduck_rl.git
uv sync
```

The virtual environment, downloaded ONNX policies, logs, and credentials are not
stored in this fork. Restore the exact VelStand policy used on the original Mac
with this pinned Hugging Face download:

```bash
uv run python - <<'PY'
from pathlib import Path
import hashlib
from huggingface_hub import hf_hub_download

path = Path(hf_hub_download(
    repo_id="pollen-robotics/microduck-policies",
    filename="velstand.onnx",
    revision="665ae4eb20bfe13bca1ef4ea6d4bb6b4a096e448",
    local_dir="policies",
))
expected = "2fc7f8ee6acd7d0d3f66a99927b5d5a56589aeed02ec714de329b252d5ccf492"
actual = hashlib.sha256(path.read_bytes()).hexdigest()
if actual != expected:
    raise RuntimeError(f"Policy checksum mismatch: {actual}")
print(f"Verified {path}")
PY
```

That pinned file was verified against the original Mac's local policy on
2026-10-12. It matches the official Hub `v8` revision at that time. The original
simulation source checkout was commit `b02fdea4` on `develop`; the learning
documents are added on top of that checkout.

On a Mac, start the policy viewer at zero movement command:

```bash
.venv/bin/mjpython scripts/infer_policy.py --walking policies/velstand.onnx --new-cmd-obs
```

Alternatively, `Launch Walking.command` starts the same viewer with a 0.3 m/s
forward command. Click the launching terminal to send controls: arrows for
forward/backward and strafing, A/E for turning, Space for zero movement command,
and Q to quit. T pauses policy updates and holds targets; it is not an emergency
stop.

This reproduces the policy artifact and dependency recipe, not a guarantee that
every laptop/OS has compatible graphics and drivers. The Mac CPU viewer is
separate from the NVIDIA/CUDA environment required for parallel RL training.
Other operating systems should follow the upstream viewer/platform instructions.

Push learning changes from either laptop with normal Git commits and `git push`.
Before switching laptops, push the current work; on the other machine use
`git pull --ff-only` while the working tree is clean. `origin` is the personal
fork and `upstream` is Pollen Robotics; review upstream changes deliberately
rather than mixing a simulator update into a measurement experiment.
