# PhysClaw-0: A Symbiotic Agentic System for Robot Autonomy via Language Corrections

[Project Page](https://open-gigaai.github.io/PhysClaw)
[Arxiv](#)

## Abstract
Autonomous data collection governs both the volume and quality of real-world trajectories available for manipulation policy learning. Existing pipelines reduce human effort through self-resetting mechanisms, VLM-based verification, or language-guided correction. However, in long-horizon collection, the same failure modes recur across episodes. A correction scoped to the episode at hand must be re-issued at every recurrence, so the cost of oversight grows with session duration rather than with the number of distinct problems. We present PhysClaw-0, a human-robot symbiotic agentic system in which the robot improves under natural-language guidance. The operator supplies corrective knowledge in natural language, and every correction is retained for reuse in all subsequent rounds. As this knowledge accumulates, a failure mode that has been corrected once is typically handled automatically thereafter, so the operator’s attention is drawn to new problems rather than to recurrences of old ones. PhysClaw-0 formulates the collection loop as a verification-gated decision process, in which the system collects, verifies, and resets autonomously and pauses to notify a remote operator only when a phase fails verification repeatedly, a boundary set by an explicit retry budget. The operator describes the observed problem in natural language. An LLM parser translates each utterance into a structured system adjustment and stores it in a Corrective Memory consulted on subsequent rounds, so that an addressed failure mode is less likely to require a second correction under the same conditions. On a real-robot desktop-clearing testbed, PhysClaw-0 matches the episode collection success rate of full human teleoperation while reducing human working time to 16% of that required by teleoperation. Language corrections repair both verifier criteria and execution strategies, improving the verifier’s agreement with human labels in all four evaluated settings and raising average single-attempt collection success from 12.5% to 47.5%, with a separate arm-selection correction improving it from 20.0% to 50.0%. Closing the loop to deployment, policies fine-tuned on PhysClaw-0-collected data match the policy success rate of those trained on full teleoperation data while requiring only a fraction of the human working time during collection.

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
