# Experiment Plan

## Phase 0 - Environment Sanity Check

Goal: verify RoboCasa can provide everything needed before any VLM work.

Tasks:

- OpenDrawer
- CloseDrawer

Checklist:

1. Create the environment.
2. Reset deterministically from a seed.
3. Render RGB.
4. Render depth.
5. Read robot base pose.
6. Read target fixture / interaction target pose.
7. Reset / place the robot at a specified SE(2) pose.
8. Save an observation and metadata record.

Deliverables:

- `scripts/smoke_test_robocasa.py`
- one saved RGB image
- one saved depth array / visualization
- one JSON metadata file

## Phase 1 - Candidate Pose Generator

Goal: produce valid last-mile candidate poses.

Initial settings:

- N = 8 candidates
- target-centered sampling
- collision filtering
- candidate yaw toward target

Visualize candidate locations from a top-down debug view if available.

Deliverables:

- reusable candidate-generation module
- candidate visualization script
- unit tests for coordinate transforms

## Phase 2 - Downstream Policy Validation

Before collecting pose labels, evaluate the fixed manipulation policy from its canonical / demonstration initialization.

Measure per task:

- canonical-pose success rate
- rollout duration
- reset reliability

If canonical performance is poor, do not collect large pose datasets yet.

## Phase 3 - Semantic Sensitivity Feasibility Study

This is the critical go / no-go experiment.

Use:

    2 task pairs x 10 scenes x 8 candidates

Suggested pairs:

- OpenDrawer vs CloseDrawer
- OpenCabinet vs CloseCabinet

For the same candidate geometry, estimate the downstream success label for each instruction / task.

Outputs:

- success vector per scene-task:

      q = [q_1, ..., q_8]

- visualization of success heatmaps / candidate plots
- similarity / difference statistics between semantic task pairs

Decision rule:

Proceed to language-conditioned training only if the success distributions exhibit meaningful task-dependent differences.

## Phase 4 - Compact Dataset Collection

Start with a small dataset, not the full RoboCasa365 corpus.

Example first target:

    4 tasks x 30 scenes x 8 candidates = 960 candidate records

Optionally repeat each rollout 2-3 times on a smaller final subset.

Store only what is needed for model training:

```json
{
  "scene_id": "...",
  "seed": 0,
  "task": "OpenDrawer",
  "instruction": "Open the drawer.",
  "candidate_id": 3,
  "base_pose": [0.0, 0.0, 0.0],
  "relative_target_pose": [0.0, 0.0, 0.0],
  "rgb_path": "...",
  "depth_path": "...",
  "depth_features": {},
  "successes": 2,
  "trials": 3
}
```

Avoid storing full rollout video for every sample. Save full trajectories only for debugging / qualitative examples.

## Phase 5 - Learning Baselines

### Baseline A: geometry-only

Inputs:

- candidate distance
- yaw
- depth/free-space metric features

Model:

- small MLP

### Baseline B: frozen VLM, no language

Input RGB only (or neutral prompt).

### Baseline C: frozen VLM + language

Input RGB + instruction.

### Baseline D: frozen VLM + language + depth features

This is the first serious proposed baseline.

## Phase 6 - Parameter-Efficient Training

Add LoRA / QLoRA to the VLM.

Start with a 2B-class backbone.

Only move to 4B after the pipeline and language benefit are verified.

Track:

- trainable parameter count
- peak GPU memory
- training wall time
- inference latency

## Phase 7 - Closed-Loop Refinement

Generate local perturbations around good and bad terminal poses.

Train / define a Ready + micro-action controller.

Evaluate:

- manipulation SR before refinement
- manipulation SR after refinement
- average number of correction steps
- extra path length / time

## Phase 8 - Evaluation

### Main metric

- Manipulation Success Rate

### Additional metrics

- Top-1 successful-candidate rate
- oracle-candidate SR
- regret to oracle candidate
- distance to success region
- closed-loop correction count
- inference latency

### Required language tests

- correct instruction
- shuffled instruction
- semantically paired / counterfactual instruction

### Generalization

Split scenes so final evaluation contains unseen layouts / styles whenever possible.

## Comparison Table Skeleton

| Method | Language | Point Cloud / 3D Scene | Task-specific Training | Closed Loop | Manipulation SR |
|---|---:|---:|---:|---:|---:|
| Random feasible | No | No | No | No | |
| Front / fixed distance | No | No | No | No | |
| Reachability | No | Geometric only | No | No | |
| Navi2Gaze-style | Yes | Method-dependent | No / zero-shot | Mostly one-shot | |
| N2M | No | Yes | Rollout-trained | No | |
| Mobi-pi | No | 3DGS | Policy-aware | No | |
| UniLM-Nav | Yes | Geometry-aware 3D lifting | Zero-shot | One-shot pipeline | |
| Ours, frozen | Yes | No explicit point cloud | Small head | No | |
| Ours, LoRA | Yes | No explicit point cloud | PEFT | No | |
| Ours, LoRA + refine | Yes | No explicit point cloud | PEFT | Yes | |
