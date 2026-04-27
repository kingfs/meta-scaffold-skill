# Meta Scaffold Skills

AI-native Go backend and CLI scaffold skills for Codex-style agents.

GitHub description:

> AI-native Codex skills for scaffolding production Go backend services and Go CLI tools with structured outputs, dry-run workflows, and agent-friendly project boundaries.

## What Is Included

This repository provides two skills under `.agents/skills/`:

- `go-backend-meta-scaffold`: scaffold or refactor production Go backend services with protobuf as the interface source of truth, gRPC/REST generation, ent, wire, cobra, viper, go-task, Docker, GitLab CI, and AI-native service operation commands.
- `go-cli-meta-scaffold`: scaffold or refactor production Go CLI tools with cobra, optional viper config, structured JSON/NDJSON output, dry-run and no-input automation paths, structured errors, go-task, golangci-lint, GoReleaser, and GitLab CI.

Both skills are designed to avoid reverse-engineering the project structure after generation. They make source/generated boundaries, command contracts, documentation, validation, and AI collaboration rules explicit from the start.

## Install

Install the skills from this GitHub repository with the same `skills add` style used by other public skill collections:

```bash
npx skills add kingfs/meta-scaffold-skill
```

or with Bun:

```bash
bunx skills add kingfs/meta-scaffold-skill
```

After installation, the agent should discover:

```text
go-backend-meta-scaffold
go-cli-meta-scaffold
```

## Usage

Ask the agent for the scaffold you want, for example:

```text
Use go-cli-meta-scaffold to create a production Go CLI named devopsctl.
It should support JSON output, dry-run, no-input automation, GitLab CI, and GoReleaser.
```

```text
Use go-backend-meta-scaffold to create a production Go backend service named order-service.
Use protobuf as the API source of truth, generate gRPC and REST, use ent for persistence,
and include Docker, GitLab CI, migration commands, and e2e tests.
```

The backend skill uses a two-phase flow when generating a new project:

```text
phase1 -> generate:ent -> phase2 -> generate -> migrate
```

This keeps ent-generated code available before runtime code that imports it is written.

## AI-native CLI Defaults

The scaffold guidance and templates bias new projects toward agent-friendly command surfaces:

- Machine-readable `--format json` output.
- `--dry-run` for commands with side effects.
- `--no-input` automation paths that fail clearly instead of prompting.
- stdout/stderr separation.
- Structured success and error envelopes.
- Explicit `trace_id`, mutation state, warnings, and next-step hints.
- Clear source/generated file boundaries.
- Project-level `AGENTS.md`, `ARCHITECTURE.md`, and `README.md` in generated projects.

## Repository Layout

```text
.agents/skills/
├── go-backend-meta-scaffold/
│   ├── SKILL.md
│   ├── assets/templates/
│   ├── checklists/
│   ├── references/
│   └── scripts/init_scaffold.sh
└── go-cli-meta-scaffold/
    ├── SKILL.md
    ├── assets/templates/
    ├── checklists/
    ├── references/
    └── scripts/init_scaffold.sh
```

## Validate Locally

Check the scaffold scripts:

```bash
bash -n .agents/skills/go-cli-meta-scaffold/scripts/init_scaffold.sh
bash -n .agents/skills/go-backend-meta-scaffold/scripts/init_scaffold.sh
```

Generate a temporary Go CLI scaffold:

```bash
.agents/skills/go-cli-meta-scaffold/scripts/init_scaffold.sh \
  /tmp/meta-scaffold-cli \
  example.com/acme/devopsctl \
  DevOpsCtl \
  sync
```

Generate a temporary Go backend scaffold:

```bash
.agents/skills/go-backend-meta-scaffold/scripts/init_scaffold.sh phase1 \
  /tmp/meta-scaffold-backend \
  example.com/acme/order-service \
  order-service \
  order.v1 \
  OrderService \
  v1 \
  Order
```

## License

Apache-2.0.
