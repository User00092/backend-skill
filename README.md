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

## Adding to Your Agent

You can have your agent install this skill using the following prompt.

```bash
Install the backend engineering agent skills from:
https://github.com/User00092/backend-skill

Read the README.md and follow its installation instructions.

Before installing, ask whether to install the skill globally (available across projects) or locally (available only within the current project). Default to global installation if the user has no preference.

Discover the correct skills directory for the current agent environment. Install the contents of `skills/backend-engineering/`, preserving `SKILL.md`, the `references/` directory, and all relative paths.

If already installed, check for updates and preserve local modifications. Do not overwrite unrelated files.

Validate that:
- SKILL.md contains valid Agent Skills YAML frontmatter.
- The declared skill name matches its directory.
- All reference links resolve.
- The skill is discoverable by the current agent.
- The agent can selectively load relevant references.

Do not copy the archive unless explicitly requested.

After installation, use this skill automatically when relevant to backend design, implementation, debugging, performance optimization, security, scalability, or code review. Load only the references relevant to the current task.

Report the installation scope, destination, version or commit, and validation results.
```

The topic files preserve their corresponding sections from the supplied original. Supplemental background and source descriptions remain in `archive/backend-skills-original.md`. This is an engineering reference, not an assertion that every video claim was independently verified.
