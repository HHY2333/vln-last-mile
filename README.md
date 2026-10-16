# VLN Last-Mile Research Context Bundle

This bundle is designed to be copied into the root of a Codex-driven research repository.

## Files

- `AGENTS.md` - persistent project constraints and research direction
- `docs/RESEARCH_CONTEXT.md` - problem framing, novelty boundaries, and risks
- `docs/METHOD.md` - current model design
- `docs/EXPERIMENT_PLAN.md` - staged implementation / experiment plan
- `docs/LITERATURE_MAP.md` - concise paper map for Codex and collaborators
- `refs/references.bib` - BibTeX database
- `refs/README.md` - reference-management workflow
- `papers/README.md` - where to place local PDFs
- `CODEX_START_PROMPT.md` - first prompt to give Codex

## Suggested Repository Layout

```text
vln-last-mile/
├── AGENTS.md
├── README.md
├── CODEX_START_PROMPT.md
├── docs/
│   ├── RESEARCH_CONTEXT.md
│   ├── METHOD.md
│   ├── EXPERIMENT_PLAN.md
│   └── LITERATURE_MAP.md
├── refs/
│   ├── references.bib
│   └── README.md
├── papers/
│   └── README.md
├── configs/
├── scripts/
├── src/
└── tests/
```

## Recommended Reference Strategy

Version-control `refs/references.bib` and `docs/LITERATURE_MAP.md`.

Keep PDFs local in `papers/` or manage them with Git LFS. Exact paper claims should be checked against the original PDFs / official repositories.

## Immediate Next Step

Use the prompt in `CODEX_START_PROMPT.md` and implement only RoboCasa Phase 0 before adding any VLM training code.
