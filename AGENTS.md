# Project Instructions: Language-Conditioned Last-Mile Navigation

## Mission

This repository studies manipulation-aware last-mile navigation for mobile manipulation in RoboCasa / RoboCasa365.

The core goal is to select and refine a robot base pose that maximizes downstream manipulation success:

    RGB + Depth + Language Instruction + Robot / Pose Features
        -> manipulation-ready candidate score
        -> local closed-loop refinement
        -> fixed downstream manipulation policy

Main research question:

> Can task-conditioned manipulation-ready navigation be learned without explicit point clouds or full 3D scene reconstruction, using a pretrained VLM, lightweight metric depth cues, manipulation-rollout supervision, and parameter-efficient adaptation?

## Non-negotiable Design Constraints

- Do not make point clouds the primary representation.
- Do not introduce PointNet / PointNet++ / Point-BERT as the main method.
- Depth is allowed and encouraged, but should remain a depth image or lightweight metric feature representation.
- Prefer 2D / 2.5D representations.
- Do not ask a VLM to directly regress centimeter-accurate `(x, y, yaw)` in the first implementation.
- Prefer candidate scoring / ranking over direct continuous pose regression.
- Prefer LoRA / QLoRA or small heads over full VLM fine-tuning.
- Avoid retraining a large VLA manipulation policy unless later experiments make it necessary.
- Use the same fixed downstream manipulation policy when comparing base-pose-selection methods.
- Keep the first implementation small and testable before scaling.

## Target Compute

Primary target: one RTX 3090 / 4090 class 24 GB GPU.

Engineering choices should remain feasible on this class of hardware unless an experiment is explicitly marked as optional / high-resource.

## Primary Environment

Use RoboCasa / RoboCasa365.

Initial semantic task pairs should include tasks where the same or similar fixture may require different interaction geometry:

- OpenCabinet / CloseCabinet
- OpenDrawer / CloseDrawer
- OpenFridge / CloseFridge

Do not begin with all 365 tasks.

## Initial Method

### 1. Candidate Pose Generation

Generate approximately 8 collision-free SE(2) candidate base poses around the manipulation target:

    p_i = (x_i, y_i, yaw_i)

Use simulator geometry / collision APIs initially. Do not train a candidate generator in Phase 0.

### 2. Candidate Observation

For every candidate pose, obtain:

- RGB observation
- depth observation
- candidate-to-target relative distance / yaw
- minimal robot embodiment information if needed

Prefer independent candidate scoring during training:

    score_i = F(I_i, D_i, instruction, robot_info, pose_info)

This is preferred over packing all candidate images into one huge VLM context for the first version.

### 3. Semantic Branch

Use a small pretrained VLM as the semantic / task-conditioned visual encoder.

Initial model family: a 2B-class VLM; 4B is a later scale-up experiment if resources allow.

The semantic branch should capture:

- manipulation intent
- target / fixture semantics
- visibility
- interaction direction
- task-dependent spatial preference

### 4. Depth / Geometry Branch

Start with lightweight metric cues before a learned depth encoder:

- target median depth
- nearest obstacle / free-space depth
- left / right free-space statistics
- candidate-target distance
- relative yaw

Optional later version: a small CNN / ViT depth encoder.

Do not reconstruct an explicit global 3D scene for the main method.

### 5. Prediction Heads

Primary output:

    q_hat_i = P(manipulation_success | I_i, D_i, L, p_i)

Secondary output:

    r_hat_i = P(manipulation_ready | I_i, D_i, L, p_i)

Optional later output:

    {left, right, forward, back, rotate_left, rotate_right, ready}

for local terminal refinement.

### 6. Closed-Loop Refinement

After reaching the best-ranked candidate:

    observe -> predict readiness / correction -> small move -> observe again

Suggested local actions for the first implementation:

- +/- 5 cm translation
- +/- 5 deg rotation
- READY

The closed-loop terminal stage is an important intended distinction from one-shot pose-selection methods.

## Supervision

The preferred label is downstream manipulation outcome, not distance from a demonstration pose:

    y_i = Success(fixed_manipulation_policy, candidate_pose_i)

If feasible, repeat candidate rollouts and store:

    q_i = successes / trials

A demonstration pose may be used as a convenient positive prior, but should not be treated as the unique optimal ground truth.

## Training Order

1. Frozen VLM + small success / ranking head.
2. Add lightweight depth metric features.
3. Add a learned depth branch if useful.
4. LoRA / QLoRA adaptation of the VLM.
5. Add Ready / micro-action refinement head.

Avoid full VLM fine-tuning in the initial project.

## Losses

Start with a success-probability objective such as BCE.

Optional pairwise ranking loss:

    L = L_BCE + lambda * L_rank

where a candidate with higher empirical manipulation success should receive a higher model score.

## Evaluation

Primary metric:

- downstream Manipulation Success Rate (MSR)

Also consider:

- Top-1 successful-candidate rate
- regret to the oracle candidate
- distance to the nearest successful region
- reposition count
- number of closed-loop refinement steps
- inference latency

Required ablations:

- RGB only
- RGB + Language
- RGB + Depth
- RGB + Depth + Language
- frozen VLM vs LoRA-adapted VLM
- one-shot vs closed-loop refinement

Required language sanity test:

- correct instruction vs shuffled / counterfactual instruction

The project must demonstrate that language contributes beyond visual geometry.

## Baselines

Simple baselines:

- Random feasible
- Front / fixed-distance pose
- Nearest feasible pose
- Reachability + collision checking

Research baselines / related methods:

- Navi2Gaze
- MoTo
- UniLM-Nav
- N2M
- Mobi-pi
- MoMa-Kitchen / NavAff
- FloAff-Kitchen

Do not claim that the proposed module directly beats the full RoboCasa365 leaderboard unless the task protocol is actually identical.

## Critical Feasibility Gate

Before large-scale training, run a small semantic sensitivity experiment:

    2 semantic task pairs
    x 10 scenes
    x 8 candidate poses

Example pairs:

- OpenDrawer vs CloseDrawer
- OpenCabinet vs CloseCabinet

Generate actual manipulation-success maps under the same candidate poses.

Only scale the language-conditioned direction if different instructions produce meaningfully different preferred-pose / success distributions.

## Engineering Order

1. Install and validate RoboCasa.
2. Load one task and render RGB + depth.
3. Load or verify a fixed downstream manipulation policy.
4. Verify high success from its canonical / demonstration pose.
5. Implement candidate pose generation.
6. Implement automated candidate rollout evaluation.
7. Export a compact RGB/depth/metadata dataset.
8. Train a frozen-VLM candidate success baseline.
9. Add depth cues.
10. Add LoRA.
11. Add closed-loop refinement.
12. Evaluate on unseen scenes / layouts.

## Documentation and Literature Rules

- Read `docs/RESEARCH_CONTEXT.md`, `docs/METHOD.md`, `docs/EXPERIMENT_PLAN.md`, and `docs/LITERATURE_MAP.md` before substantial methodological changes.
- Use citation keys from `refs/references.bib` in research notes.
- Do not invent paper claims. If exact implementation details are needed, consult the original PDF / project page.
- Local paper PDFs may be placed under `papers/`; see `papers/README.md`.
