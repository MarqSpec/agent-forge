# Guide for coding agents

This repository is **AgentForge**, a .NET clinical copilot sidecar for OpenEMR. Before changing code, read
the docs that match your task; they are the contract the implementation and tests cite.

## Documentation map

| Document | Use it for |
|---|---|
| [README.md](README.md) | What the product does, how to run the stack locally, and a short repo layout |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Core copilot components, trust boundaries, verification, and architecture decisions |
| [ARCHITECTURE-DOCUMENTS.md](ARCHITECTURE-DOCUMENTS.md) | Document ingestion, hybrid retrieval, evidence agents, and the Postgres data tier |
| [CONVENTIONS.md](CONVENTIONS.md) | Language, dependencies, C# style, HTTP/resilience, logging, tests, and security rules |
| [DEPLOYMENT.md](DEPLOYMENT.md) | Configuration, Docker Compose, bootstrap, images, smoke checks, and rollback |
| [INTERFACES.md](INTERFACES.md) | SMART/OAuth and FHIR with OpenEMR; sidecar HTTP, SignalR, and MCP tool contracts |
| [REQUIREMENTS.md](REQUIREMENTS.md) | Users, use cases (`UC-`), and functional (`FR-`) / non-functional (`NFR-`) requirements |
| [METRICS.md](METRICS.md) | Key metrics, SLOs, dashboards, alerts, and how baselines were measured |
| [PROMPTS.md](PROMPTS.md) | Which components call the LLM and what is enforced in code instead of prompt text |
| [SECURITY-PLATFORM.md](SECURITY-PLATFORM.md) | The adversarial security platform, threat categories, and how it relates to the copilot |
| [evals/README.md](evals/README.md) | Golden eval cases and the deterministic regression gate |
| [observability/README.md](observability/README.md) | Optional Prometheus, Loki, Tempo, and Grafana stack |
| [reverse-proxy/README.md](reverse-proxy/README.md) | The nginx same-origin front door (`/` → OpenEMR, `/agentforge` → sidecar) |
| [security-platform/README.md](security-platform/README.md) | The security replayer image, `serve` vs `run`, and allow-listed targets |
| [tests/bruno-collection/README.md](tests/bruno-collection/README.md) | Runnable Bruno requests against core HTTP endpoints |

Prompt **text** lives in source constants; [PROMPTS.md](PROMPTS.md) indexes them—do not treat the markdown as
the authoritative copy.

## Build, test, and lint

Run these from the repository root (same commands as [README.md](README.md) § *Build and test*):

```bash
dotnet build AgentForge.slnx
dotnet format --verify-no-changes
dotnet test tests/AgentForge.UnitTests
dotnet test tests/AgentForge.EvalTests
dotnet run --project tests/AgentForge.Evals -- evals
dotnet test tests/AgentForge.HermeticTests
dotnet test tests/AgentForge.IntegrationTests
dotnet test tests/AgentForge.IntegrationTests --filter "Deployment=None"
sh security-platform/run-selftest.sh
```

Integration tests that need a deployed stack are configured via environment variables; see
[CONVENTIONS.md](CONVENTIONS.md) §8.2. Prefer the unit, eval, hermetic, and `Deployment=None` integration
suites when you cannot reach OpenEMR, Postgres, or a live LLM.

## How to work here

- **Keep changes small and scoped** to the task; match existing patterns in the nearest code and tests.
- **Follow [CONVENTIONS.md](CONVENTIONS.md)** for C#, dependencies, async/cancellation, errors, and test tiers.
- **Never commit secrets**—API keys, passwords, tokens, or private URLs belong in environment variables or
  local `.env` (see [.env.example](.env.example)), not in source or logs.
- **Synthetic data only**—demo patients and fixtures; do not add or process real PHI.
- **Preserve stable IDs** when touching requirements, metrics, architecture decisions, or prompt contracts
  referenced from code and tests.
- **Do not re-document** what the linked files already specify; link to the section you relied on instead.
