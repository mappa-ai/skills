# Mappa skills

Monorepo of **Agent Skills** for development workflows and Mappa product integrations. Skills are packaged as directories containing `SKILL.md` (agent instructions) and, for humans browsing the repo, `README.md` (detailed overview).

## Layout

| Directory | Purpose |
|-----------|---------|
| [utils-skills/](utils-skills/) | Cross-cutting skills for planning, execution, ecosystem discovery, and tooling (e.g. PRs, CodeRabbit). |
| [internal-skills/](internal-skills/) | Mappa-internal product skills (e.g. Conduit SDK integration). |

## Skills index

### Utils

| Skill | Summary | Human doc |
|-------|---------|-----------|
| `find-skills` | Discover and vet skills on skills.sh; install via `npx skills`. | [utils-skills/find-skills/README.md](utils-skills/find-skills/README.md) |
| `grill-me` | One-question-at-a-time design stress-test with recommendations. | [utils-skills/grill-me/README.md](utils-skills/grill-me/README.md) |
| `write-a-skill` | Author new skills: structure, descriptions, progressive disclosure. | [utils-skills/write-a-skill/README.md](utils-skills/write-a-skill/README.md) |
| `handle-coderabbit` | Triage and fix CodeRabbit PR comments with `gh`, validate, commit. | [utils-skills/handle-coderabbit/README.md](utils-skills/handle-coderabbit/README.md) |
| `do-work` | End-to-end unit of work: implement, typecheck/test, commit. | [utils-skills/do-work/README.md](utils-skills/do-work/README.md) |
| `prd-to-plan` | Turn a PRD into phased tracer-bullet plans under `./plans/`. | [utils-skills/prd-to-plan/README.md](utils-skills/prd-to-plan/README.md) |
| `write-a-prd` | Interview + explore repo, then write a PRD under `plans/`. | [utils-skills/write-a-prd/README.md](utils-skills/write-a-prd/README.md) |
| `refine-linear-issue` | Turn rough descriptions into structured Linear issues using codebase context. | [utils-skills/refine-linear-issue/README.md](utils-skills/refine-linear-issue/README.md) |

### Internal

| Skill | Summary | Human doc |
|-------|---------|-----------|
| `conduit-sdk` | Integrate Mappa Conduit SDKs (TS, Python, Go, Rust); webhooks-first. | [internal-skills/conduit-sdk/README.md](internal-skills/conduit-sdk/README.md) |

## Install with `npx skills`

Replace the URL with your fork if needed. The `--skill` value is the **path from the repo root** to the skill directory.

```bash
# Example: Conduit SDK
npx skills add https://github.com/mappa-ai/skills --skill internal-skills/conduit-sdk

# Example: a util skill
npx skills add https://github.com/mappa-ai/skills --skill utils-skills/do-work
```

**Breaking change:** Older docs used `--skill conduit-sdk` when the skill lived at the repository root. After this layout change, use `internal-skills/conduit-sdk` (or vendor the folder under the name your tool expects).

## Validate locally

CI uses `uv` and `agentskills validate`. Equivalent local check:

```bash
uvx --from skills-ref agentskills validate internal-skills/conduit-sdk
# Optional: validate each util skill
for d in utils-skills/*/; do uvx --from skills-ref agentskills validate "${d%/}"; done
```

## Contributing

Add new skills under the appropriate subtree, include both `SKILL.md` and `README.md`, and extend the tables in this file.
