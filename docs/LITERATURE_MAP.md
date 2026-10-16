# Literature Map

Use this file as a compact map. For exact numerical claims, equations, architecture details, or reproduction, consult the original paper / official code.

## Core manipulation-aware last-mile / base-placement papers

### `zhang2025momakitchen` - MoMa-Kitchen

**Role in this project:** manipulation-aware last-mile benchmark and NavAff baseline.

Key relevance:

- frames last-mile navigation as choosing floor regions that enable downstream manipulation
- uses RGB-D and explicit point-cloud processing in the baseline
- provides manipulation-derived affordance labels

Use it to justify evaluating downstream manipulation success rather than navigation proximity alone.

### `chai2026n2m` - N2M

**Role in this project:** strongest conceptual source for rollout-grounded policy preference.

Key relevance:

- learns preferable initial base poses from actual downstream manipulation-policy rollouts
- demonstrates that geometric reachability alone is insufficient for learned manipulation policies
- predicts a distribution over preferred poses
- uses egocentric RGB point clouds and Point-BERT

Our intended difference: keep rollout-grounded supervision but replace the explicit point-cloud representation with RGB + language + lightweight depth / pose cues.

### `zhong2026floaff` - FloAff-Kitchen

**Role in this project:** task-specific floor-affordance learning and strong learned placement baseline.

Key relevance:

- explicitly separates shared placement priors from task-specific interaction constraints
- evaluates multiple manipulation skills
- remains point-cloud-based and closed-set / task-specific rather than open-vocabulary language-conditioned at inference

### `yang2025mobipi` - Mobi-pi

**Role in this project:** policy-aware base-pose optimization without language.

Key relevance:

- policy mobilization formulation
- scores candidate poses according to compatibility with a fixed manipulation policy
- uses 3D Gaussian Splatting / novel-view synthesis and optimization

Our intended difference: much lighter 2D/2.5D representation and language conditioning.

### `shan2026dockanywhere` - DockAnywhere

**Role in this project:** complementary robustness-to-docking-error work.

Key relevance:

- augments manipulation demonstrations across docking-point variation
- targets viewpoint / docking generalization of manipulation policy rather than selecting the best language-conditioned base pose

Use it to motivate residual docking-error robustness and possible future joint pose-selection + policy-robustness studies.

## VLM / language-conditioned navigation-to-manipulation papers

### `zhu2024navi2gaze` - Navi2Gaze

**Role in this project:** very direct VLM candidate-pose reasoning baseline.

Key relevance:

- uses task descriptions and a VLM to score / select navigation poses
- emphasizes target orientation / gazing and commonsense spatial reasoning

Novelty warning: “VLM + language + candidate pose ranking” alone is not sufficient novelty.

### `wu2025moto` - MoTo

**Role in this project:** direct zero-shot VLM-based mobile manipulation navigation baseline.

Key relevance:

- uses VLM-generated interaction keypoints
- generates docking points for fixed-base manipulation foundation models
- zero-shot interaction-aware navigation

### `zhang2026unilmnav` - UniLM-Nav

**Role in this project:** closest high-level problem formulation.

Key relevance:

- zero-shot open-vocabulary last-mile navigation
- uses an MLLM for view selection, task-conditioned affordance grounding, and geometry-aware base-pose reasoning

Novelty warning: the project must differentiate through representation / supervision / adaptation / closed-loop refinement, not merely “language-conditioned last-mile navigation.”

## Environment / benchmark

### `nasiriany2026robocasa365` - RoboCasa365

**Role in this project:** primary simulation environment and task source.

Key relevance:

- 365 household kitchen tasks
- 2,500 diverse kitchen environments
- large-scale demonstrations
- suitable atomic tasks for semantic pairs such as open / close variants

The project should initially use a small subset rather than attempting full benchmark training.

## Reading Priority

1. N2M
2. UniLM-Nav
3. MoTo
4. Navi2Gaze
5. FloAff-Kitchen
6. MoMa-Kitchen
7. Mobi-pi
8. DockAnywhere
9. RoboCasa365 benchmark paper / docs
