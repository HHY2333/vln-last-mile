# Reference Management

## Recommended Workflow

Keep bibliographic metadata under version control:

    refs/references.bib

Keep paper PDFs separately under:

    papers/

The `papers/` directory is intentionally ignored by Git in this bundle because PDFs are large and may have redistribution / storage concerns. If your group wants PDFs versioned, use Git LFS instead.

## Suggested Local PDF Names

- `2025_MoMaKitchen.pdf`
- `2026_N2M.pdf`
- `2026_FloAffKitchen.pdf`
- `2026_DockAnywhere.pdf`
- `2024_Navi2Gaze.pdf` (or the final IROS version)
- `2025_MoTo.pdf`
- `2025_MobiPi.pdf`
- `2026_UniLMNav.pdf`
- `2026_RoboCasa365.pdf`

## How Codex Should Use References

For general project context, `docs/LITERATURE_MAP.md` is enough.

For exact reproduction details, ask Codex to read the relevant local PDF / official repository before implementing a baseline. Example prompt:

> Before implementing the N2M baseline, read `docs/LITERATURE_MAP.md`, the `chai2026n2m` BibTeX entry, and `papers/2026_N2M.pdf`. Summarize the exact input representation, training supervision, output parameterization, candidate safety filtering, and evaluation protocol. Then implement only the parts required for our comparison.

## Citation Keys

Use the BibTeX keys in `references.bib` consistently in notes, LaTeX, and experiment docs.

Examples:

    \cite{chai2026n2m}
    \cite{zhang2026unilmnav,wu2025moto}

## Important

Do not let Codex infer precise paper claims from filenames or from the literature map alone. Exact numbers and implementation details should be checked against the paper or official code.
