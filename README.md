# Backend Agent Skills

A Git-ready, progressively-loaded [Agent Skills](https://agentskills.io/specification) repository converted from the original 21-topic `backend-skills.md`.

## Layout

```text
backend-agent-skills/
├── README.md
├── .gitignore
├── archive/
│   └── backend-skills-original.md   # Lossless original combined document
└── skills/
    └── backend-engineering/
        ├── SKILL.md                 # Main discovery metadata, workflow, topic router
        └── references/
            ├── 01-debounce-and-cancel-stale-requests.md
            └── ...                  # 21 focused topic references
```

## Install / use

Point a skills-compatible agent at `skills/backend-engineering/` or copy that folder into its supported skills location (for example, `.agents/skills/backend-engineering/` in the project when supported). The agent discovers `SKILL.md` first and reads referenced files only when needed. The full provenance archive remains available in this repository.

## Git

This package is a **local initialized Git repository**, not a published GitHub repository. To publish it yourself:

```bash
git remote add origin https://github.com/OWNER/REPO.git
git push -u origin main
```

The topic files preserve their corresponding sections from the supplied original. Supplemental background and source descriptions remain in `archive/backend-skills-original.md`. This is an engineering reference, not an assertion that every video claim was independently verified.
