# ADR-0001: Record architecture decisions

- Status: accepted
- Deciders: Ma Yitong, Pan Yaqin, Жанарбек Серлан, Бақытбай Бұлдірген
- Date: 2026-11-17

## Context and Problem Statement

Our team designs the Orders and Delivery parts of a food-delivery platform. Team members
change tasks often, and decisions made in chat or in meetings are quickly forgotten.
We need a short, reviewable record of each important technical decision and of why we made it.

## Decision Outcome

We will record every architecturally significant decision as an Architecture Decision Record (ADR)
in `docs/adr/`, using the MADR template (https://adr.github.io/madr/).

- Files are named `NNNN-short-title.md` and numbered in order.
- Every ADR is added through a pull request and needs at least one review comment before merge.
- An accepted ADR is never edited in substance. If a decision changes, we write a new ADR and set
  the old one to `superseded by ADR-NNNN`.
- `docs/adr/README.md` lists every ADR with its number, title and status.

### Consequences

- Good, because new members can read why the system looks the way it does.
- Good, because decisions are reviewed like code.
- Bad, because writing an ADR takes about one hour (mitigation: only for decisions that are hard to reverse).
