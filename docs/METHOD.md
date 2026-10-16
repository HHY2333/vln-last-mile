# Proposed Method

## 1. Overview

The initial model scores candidate robot base poses independently.

For candidate pose `p_i`:

    RGB_i + Depth_i + Language + Pose/Robot Features
        -> semantic feature + geometry feature
        -> fusion
        -> manipulation success score

The best candidate is selected, the robot navigates to it, and a closed-loop terminal stage optionally performs small corrections before manipulation.

## 2. Candidate Pose Set

Let the candidate set be:

    P = {p_1, ..., p_N},  p_i = (x_i, y_i, yaw_i)

Initial value:

    N = 8

Candidate generation should initially use simulator target geometry and collision checks. The first research question is pose selection, not target localization.

Possible first sampling pattern:

- 2 radii around the target
- 4 angular sectors
- yaw initialized toward the target / interaction region

Candidate generation code should be separated from model code so later sampling strategies can be swapped in.

## 3. Inputs

### RGB

Candidate egocentric RGB observation `I_i`.

### Depth

Start with compact derived features instead of a large learned depth model:

    g_depth = [
        target_median_depth,
        nearest_depth,
        left_free_space,
        right_free_space,
        candidate_target_distance,
        relative_yaw
    ]

A learned depth-image encoder can be an ablation / extension.

### Language

Natural-language task instruction `L`.

Use semantically paired instructions in the dataset design.

### Robot / Candidate Features

Minimal optional information:

- arm effective reach
- base width / footprint
- candidate distance to target
- relative yaw

Avoid overloading the model with privileged simulator state in the main reported method.

## 4. Semantic Encoder

Initial backbone: a small VLM.

Recommended progression:

1. frozen 2B-class VLM feature extractor
2. LoRA / QLoRA adaptation
3. optional 4B scale experiment

For each candidate:

    z_v = F_vlm(I_i, L)

The VLM should not directly produce continuous centimeter-level pose coordinates in the initial model.

## 5. Geometry Encoder

First version:

    z_g = MLP(g_depth, pose_features, robot_features)

Optional version:

    z_d = F_depth(D_i)

where `F_depth` is a lightweight CNN / small ViT.

## 6. Fusion and Heads

Fuse:

    z_i = Fusion(z_v, z_g [, z_d])

Primary head:

    q_hat_i = sigmoid(MLP_success(z_i))

interpreted as predicted downstream manipulation success probability.

Optional Ready head:

    r_hat_i = sigmoid(MLP_ready(z_i))

Optional local-action head:

    a_t in {LEFT, RIGHT, FORWARD, BACK, ROT_L, ROT_R, READY}

## 7. Supervision

For candidate `p_i`, reset the scene, place / navigate the robot to the candidate, and execute the same downstream manipulation policy used for all selection methods.

Single rollout label:

    y_i in {0, 1}

Repeated rollout label:

    q_i = successes_i / trials_i

Repeated rollouts are preferred on the final test subset because manipulation policies may be stochastic.

## 8. Objective

Initial objective:

    L_BCE = BCE(q_hat_i, q_i)

Optional pairwise ranking objective for candidates within the same scene-task instance:

    if q_i > q_j, encourage s_i > s_j

Total:

    L = L_BCE + lambda_rank * L_rank

## 9. Inference

For all feasible candidates:

    s_i = Model(I_i, D_i, L, p_i)

Select:

    p_star = argmax_i s_i

Then navigate to `p_star`.

## 10. Closed-Loop Terminal Refinement

After reaching `p_star`, repeatedly observe the current RGB-D frame and predict either READY or a small correction.

Suggested initial action set:

- LEFT_5CM
- RIGHT_5CM
- FORWARD_5CM
- BACK_5CM
- ROTATE_LEFT_5DEG
- ROTATE_RIGHT_5DEG
- READY

Limit the number of refinement steps, e.g. 3-5, to bound latency and path overhead.

## 11. Core Ablations

- Vision-only vs Vision+Language
- Vision+Language vs Vision+Depth+Language
- metric depth features vs learned depth encoder
- frozen VLM vs LoRA
- one-shot ranking vs ranking + closed-loop refinement
- correct vs shuffled instruction
- task-seen vs task-pair / scene-unseen split

## 12. Important Protocol Rule

All base-pose-selection methods must be evaluated using the same downstream manipulation policy and the same candidate-pose validity / collision criteria whenever possible.
