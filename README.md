# PhysClaw-0

**A symbiotic agentic system for robot autonomy via language corrections.**

PhysClaw-0 collects real-robot demonstrations with a verification-gated
analyze → collect → reset loop. The system runs autonomously for most episodes
and pauses for a remote operator only when a phase exhausts its retry budget.
The operator replies in natural language; the agent parses that feedback into
plan / criterion updates and retains them in **conversation context**
(Corrective Memory — no separate store module) so the same failure mode is
handled automatically on later rounds.

Manipulation uses **SAM3 text prompt → AnyGrasp → joint grasp/place**,
orchestrated by OpenClaw-style skills, plus a post-collect stats / quality
platform.

This repository packages first-party pipelines, motion helpers, dataset
stats/quality tools, and four agent skills. It does **not** redistribute the
proprietary AnyGrasp runtime (`.so`, license, checkpoint).

> **Hardware note:** The reference lab used Agilex Piper arms and Intel RealSense
> cameras. PhysClaw-0 does **not** require that stack — configure your own arm
> driver, camera launch, ROS topics, intrinsics, and extrinsics in
> `configs/paths.env`.

## Layout

```text
PhysClaw-0/
├── skills/
├── grasp-tools/            # Recorder pipeline + collect/ stats
├── configs/                # paths.env.example, camera_extrinsics.example.json
└── docs/
```

## Quick start

1. Copy and edit config:

```bash
cp configs/paths.env.example configs/paths.env
# Set CONDA_ENV=physclaw-0, CONDA_ENV_SAM3 / CONDA_ENV_ANYGRASP (official envs),
# ARM_* / CAMERA_* / CAM_FX.. / topics / extrinsics for YOUR robot.
```

2. Environments (see [docs/setup.md](docs/setup.md)):
   - **One** project conda env: `physclaw-0` (capture, IK/motion, collect tooling)
   - Install **SAM3** and **AnyGrasp** with their official docs; set `CONDA_ENV_SAM3` / `CONDA_ENV_ANYGRASP` to those env names

3. Grasp + place:

```bash
source configs/paths.env
bash grasp-tools/run_multi_pipeline_recorder.sh --mode diligent --no-gui --top-down \
  --task "orange" right 0.3 0.27 0
```

4. Full symbiotic collector: point your agent at `skills/`
   (`self-learning` → `1-analyze-task`). Phase A plans collection/reset from
   language + scene; Phase B runs the verified loop (retry limit **N=3** per
   phase). On Alert, reply in natural language — the agent updates
   `analyze_result.yaml` / criteria and continues with that context.

## Skills

| Skill | Role |
|-------|------|
| `self-learning` | Thin orchestrator; holds `analyze_result.yaml` |
| `1-analyze-task` | LLM/VLM task analysis → collect/reset plan; verified loop + language intervention |
| `grasp-tool` | Calls `run_multi_pipeline_recorder.sh` (SAM3 → AnyGrasp → grasp/place) |
| `understand-three-view-images` | Capture RGB and VLM-describe / judge success |

## Dependencies (summary)

| Component | Notes |
|-----------|--------|
| Conda `physclaw-0` | Project code: capture, motion, collect stats |
| Official SAM3 env | Set `CONDA_ENV_SAM3` after following SAM3 install docs |
| Official AnyGrasp env | Set `CONDA_ENV_ANYGRASP` after following AnyGrasp install docs |
| Your arm + camera ROS stack | Configure launches, topics, calibration |
| VLM API (`ARK_API_KEY`) | Scene describe + collect/reset judges |
| `COLLECT_RECORDER_PY` | External HDF5 recorder |

## Acknowledgments

Thanks to [AnyGrasp](https://github.com/graspnet/anygrasp_sdk) and
[SAM 3](https://github.com/facebookresearch/sam3) for the grasp and segmentation
backends used in this pipeline.

## License

Apache License 2.0 — see [LICENSE](LICENSE). Third-party runtimes are **not**
shipped; see [NOTICE](NOTICE) and [docs/setup.md](docs/setup.md).
