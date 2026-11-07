# verse-systems-docs

Product rules, architecture decisions, RFCs and service contracts for Verse, a time-driven marketplace built around Market Days.

This repository is the source of truth for anything that spans more than one Verse service. Each service has its own repository under the `verse-systems` organisation. Cross-service decisions, product rules and RFCs (including their schemas and API contracts) live here.

## Verse in brief

Verse is a single online marketplace organised around **Market Days**: regular, platform-scheduled trading windows (24 hours by default). During a Market Day, products that sellers have selected get a dedicated section, a Market Day price, and optional price negotiation with the seller's assistant. Outside a Market Day, Verse works like a normal marketplace.

Start with the product document: [`product/verse-marketplace.md`](product/verse-marketplace.md).

## Repository layout

| Path | Purpose |
|---|---|
| `product/` | Product rules, glossary, MVP scope |
| `adr/` | Architecture Decision Records: short records of decisions already made |
| `rfcs/` | Proposals that need discussion before a decision. Each RFC contains the schemas and API contracts for the project it covers |
| `architecture/` | Service boundaries, flows, diagrams (Mermaid source in `architecture/diagrams/`) |
| `standards/` | Organisation-wide conventions (API style, error format, repo template) |
| `project-instructions.md` | Working instructions for the project |

Folders are created when they have content, so some of these may not exist yet.

## Source precedence

If two documents disagree, the first one in this list wins:

1. Accepted ADRs
2. The product document (`product/`)
3. The sequence diagram and architecture documents (`architecture/`)
4. The original Verse Backend Architecture Specification, which is superseded wherever it conflicts with the above (for example: Kafka is the message broker, not RabbitMQ; the currency is Naira; the hidden minimum price lives in `verse-market-core`)

If a conflict is not covered by this list, raise it instead of picking silently.

## Making changes

- **A decision has been made:** write an ADR. One decision per file, numbered, for example `adr/0001-session-state-machine.md`.
- **You want feedback before deciding:** write an RFC. One proposal per file, with a status of Draft, In Review, Accepted or Rejected. An RFC includes the schemas and API contracts for the project it covers. An accepted RFC usually produces one or more ADRs.
- **Small edits** (typos, clarifications that change no rule): edit the file directly.
- Use pull requests for ADRs and RFCs, even when there is only one reviewer. The review history is part of the record.
- Keep diagrams as text (Mermaid), not images, so changes can be reviewed like code.

Product documents tag each rule as `[Decided]`, `[Proposed]` or `[Open]`. Change a tag only when a decision has actually been made.

## Status

Planning phase. No services have been implemented. The product document is at v1, with a short list of open decisions at the end of it.

## Owner

Abimbola Jolayemi