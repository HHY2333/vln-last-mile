# First Codex Prompt

Copy the prompt below into Codex after placing this bundle at the repository root.

---

Read `AGENTS.md` and all Markdown files under `docs/` and `refs/README.md` before making changes.

We are starting Phase 0 only. Do not implement VLM training yet.

First inspect the repository and propose the smallest implementation plan for:

1. installing / validating RoboCasa and its current dependencies,
2. loading the `OpenDrawer` and `CloseDrawer` tasks,
3. rendering RGB and depth observations,
4. reading the robot base pose and target fixture / interaction pose,
5. resetting or placing the robot at a controlled SE(2) base pose,
6. generating 8 candidate poses around the target without point clouds,
7. collision-filtering these candidates using simulator / geometry APIs,
8. saving one candidate observation and metadata record to disk.

Constraints:

- Do not introduce point-cloud dependencies.
- Prefer small, testable scripts.
- Keep simulator-specific APIs behind helper functions so RoboCasa version changes are localized.
- Add logging and assertions for coordinate-frame assumptions.
- Do not download the full RoboCasa365 dataset.
- Do not add a large VLA / VLM dependency yet unless it is required for the environment smoke test.

After proposing the plan, implement only the environment sanity-check portion and document exact commands in `README.md`.

At the end, report:

- files changed,
- exact commands to run,
- expected outputs,
- unresolved RoboCasa API / version assumptions,
- next smallest step.
