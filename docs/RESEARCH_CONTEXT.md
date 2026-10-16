# Research Context

## 1. Problem

Conventional navigation often considers success to be reaching the vicinity of a target. Mobile manipulation requires a stronger condition: the robot must reach a terminal base pose from which a downstream manipulation policy can actually succeed.

This project focuses on the navigation-to-manipulation transition / last-mile navigation problem.

The intended formulation is:

    observation + task instruction + robot information
        -> manipulation-ready base-pose preference
        -> terminal refinement
        -> downstream manipulation

The main research question is not simply whether a VLM can choose a navigation goal. It is:

> Can a pretrained VLM, adapted with limited data and combined with lightweight depth geometry, learn a task-conditioned manipulation-success preference over base poses without explicit point-cloud reconstruction?

## 2. Why Language Matters

Existing manipulation-aware base-placement methods often receive a known target or are trained per manipulation policy / skill. The language-conditioned setting asks whether the same scene can induce different preferred base-pose distributions under different instructions.

A strong experiment therefore keeps visual context as similar as possible while changing the instruction, for example:

- Open the drawer.
- Close the drawer.

or:

- Open the cabinet.
- Retrieve an object from the cabinet.

The project should avoid a weak setup in which the task can be inferred trivially from the image alone.

## 3. Representation Choice

The project intentionally avoids explicit point clouds as the primary method.

Preferred representation:

- RGB for semantic and interaction reasoning
- depth image or compact metric depth features for distance / free-space cues
- SE(2) candidate pose features
- optional simple robot embodiment parameters

This is a 2D / 2.5D approach rather than a full 3D reconstruction approach.

## 4. Relationship to Key Existing Work

### MoMa-Kitchen / NavAff

MoMa-Kitchen formalizes affordance-grounded last-mile navigation and provides floor-affordance supervision for manipulation-aware final positioning. Its baseline explicitly uses RGB-D and point-cloud processing. This work is an important manipulation-aware baseline and dataset reference, but it is not an open-vocabulary language-conditioned base-pose method.

### N2M

N2M learns the preferred initial pose of a downstream manipulation policy from successful rollouts. Its key idea is highly relevant: pose quality should be defined by actual policy success rather than geometry alone. N2M uses egocentric RGB point clouds and learns a pose-preference distribution. This project aims to retain rollout-grounded supervision while replacing explicit point-cloud reasoning with RGB + language + lightweight depth cues.

### FloAff-Kitchen

FloAff-Kitchen models floor affordance across heterogeneous manipulation skills and explicitly studies shared vs task-specific placement constraints. It is an important comparison for task-specific affordance learning, but its inference formulation is not natural-language-conditioned and it relies on point-cloud representations.

### DockAnywhere

DockAnywhere addresses the complementary side of the interface: making manipulation policies robust to docking-point variation through demonstration augmentation. It does not primarily solve task-conditioned base-pose selection, but motivates evaluating robustness to residual docking error.

### Navi2Gaze

Navi2Gaze uses a VLM to score candidate navigation poses according to a task description and target-gazing requirements. It is a direct VLM baseline for candidate-pose reasoning. Therefore, novelty cannot be claimed merely from “using a VLM to rank poses.”

### MoTo

MoTo is a zero-shot interaction-aware navigation plug-in for mobile manipulation. It uses VLM-generated interaction keypoints and motion optimization to obtain docking points for fixed-base manipulation foundation models. It is a direct language/VLM mobile-manipulation baseline.

### UniLM-Nav

UniLM-Nav directly studies zero-shot open-vocabulary last-mile navigation. It uses an MLLM for view selection, task-conditioned affordance grounding, and geometry-aware base-pose reasoning. This is the closest high-level problem formulation. The intended distinction of this project should therefore be the combination of:

- no explicit point cloud / scene-level 3D reconstruction
- lightweight 2D / 2.5D metric geometry
- manipulation-rollout supervision
- parameter-efficient adaptation
- closed-loop terminal refinement

### Mobi-pi

Mobi-pi studies policy mobilization: finding a mobile base pose from which a fixed manipulation policy remains in-distribution and succeeds. It uses 3D Gaussian Splatting and pose scoring / optimization. It is a strong non-language, policy-aware comparison.

## 5. Intended Contribution

A defensible target contribution is:

> A lightweight, task-conditioned last-mile navigation model that predicts manipulation success over candidate base poses from RGB, language, and lightweight depth cues, trained with limited downstream-policy rollouts and refined through a closed-loop terminal controller, without explicit point clouds or scene-level 3D reconstruction.

The novelty is the combination, not any single ingredient by itself.

## 6. Main Research Risks

### Risk A: language may not change the success landscape

If Open/Close or other instruction pairs share almost identical successful base-pose regions, language conditioning will provide little measurable benefit. This must be tested before large-scale data collection.

### Risk B: manipulation-policy noise may dominate labels

If the downstream policy fails frequently even from canonical poses, candidate success labels become noisy and difficult to attribute to base placement. Canonical-pose policy success should be measured first.

### Risk C: visual task leakage

If the visual state directly reveals the task (e.g. open-door images only occur in CloseDoor episodes), a vision-only model may infer task identity. Counterfactual / instruction-shuffle evaluation is required.

### Risk D: candidate rendering may make the task too easy

Rendering a separate future view at every candidate pose is convenient in simulation but may not correspond to a deployable real robot unless the robot can actively observe from those poses. Treat this as either an explicit simulation oracle / look-ahead setting or develop a later current-view-only version.
