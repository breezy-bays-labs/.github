# Breezy Bays Labs

Tools and methodology for agentic software development — where AI agents are teammates, not assistants. One human sets direction; many concurrent agent sessions do the work.

Our public focus is **code-quality tooling**: fast, deterministic analysis that catches complexity, duplication, and weak tests before they ship.

## Projects

### Code-quality tooling

| Project | Description |
|---------|-------------|
| [crap4ts](https://github.com/breezy-bays-labs/crap4ts) | CRAP-score analyzer for TypeScript — surfaces complex, under-tested functions |
| [crap-rs](https://github.com/breezy-bays-labs/crap-rs) | CRAP-score analyzer for Rust |
| [dry-rs](https://github.com/breezy-bays-labs/dry-rs) | Structural duplication detector for Rust |
| [scrap-rs](https://github.com/breezy-bays-labs/scrap-rs) | Static test-smell detector for Rust |
| [cute-dbt](https://github.com/breezy-bays-labs/cute-dbt) | Unit-test explorer for dbt — turns `manifest.json` into a self-contained HTML report |

### Dogfooding

| Project | Description |
|---------|-------------|
| [mokumo](https://github.com/breezy-bays-labs/mokumo) | Production management for decorated-apparel shops (SvelteKit + Rust). A pet project that dogfoods the agentic-development tooling and methodology we build. |

## How we work

We adapt [Shape Up](https://basecamp.com/shapeup) for a team of one human and N concurrent agent sessions. Because agents start fresh each session, the methodology, pipeline, and conventions are the shared memory. The human keeps three irreplaceable roles — **bet** (what to build next), **interview** (domain knowledge and validation), and **verify** (does the result match intent) — and everything in between is agent-executable.
