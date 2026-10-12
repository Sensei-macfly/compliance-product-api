---
name: Collaboration workflow
description: How Replit and other agents should use GitHub as the project's source of truth.
---

The repository's `main` branch is the project's source of truth.

Before starting a new task:
1. sync with `main`;
2. read `docs/CDC_V1.md`, `docs/ARCHITECTURE.md`, `docs/DECISIONS.md`, `docs/SOURCES_APIS.md`, `docs/ROADMAP.md`;
3. read relevant files in `.agents/memory/`;
4. flag contradictions instead of silently overriding prior decisions.

A chat discussion is not a project decision until it is reflected in the reference documentation.

Use four states mentally:
- DISCUSSION
- PROPOSAL
- DECISION
- IMPLEMENTED

Do not create a PR for every conversational exchange. Batch coherent changes.

For structural changes:
- use a dedicated branch;
- update documentation before or with implementation;
- summarize assumptions and remaining open questions;
- open a PR;
- after merge, restart from `main`.

Do not reintroduce previously rejected product assumptions, especially:
- asking clients to guess FBA countries;
- treating VAT as proof of FBA/EPR eligibility;
- treating an upstream EPR number as sufficient proof of product/route coverage;
- duplicating business logic in the dashboard instead of the API.

If a current request conflicts with the docs, stop and surface the conflict before implementing it.
