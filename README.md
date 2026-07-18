# cloud-itonami-isco-7322

Open Occupation Blueprint for **ISCO-08 7322**: Printers.

This repository designs a forkable OSS business for a print-shop scheduling and logistics coordination practice: a print-shop scheduling and supply-coordination robot manages crew/task records under a governor-gated actor, so a print-shop crew keeps its own operating records instead of renting a closed workforce-management SaaS.

**Maturity: `:implemented`.** `src/pressman/` implements the
`PressmanActor` as a `langgraph.graph/state-graph`
(`pressman.actor`) wired to a `Pressman Advisor`
(`pressman.advisor`) and an independent `PressmanGovernor`
(`pressman.governor`), following the itonami actor pattern
(ADR-2607121000): `:intake -> :advise -> :govern -> :decide -+-> :commit
(:ok? true) +-> :request-approval (:escalate? true, human-in-the-loop
interrupt) +-> :hold (:hard? true)`. HARD invariants (always hold,
never overridable): worker provenance, shop provenance,
no-actuation (`:effect` must be `:propose`), a closed op-allowlist
(`:log-work-record`, `:schedule-crew-operation`,
`:flag-safety-concern`, `:coordinate-supply-order` — nothing else may
ever be proposed), and a permanent, unconditional block on any
proposal that would directly finalize a press-run-execution decision
(e.g. deciding to proceed with a specific printing press run) or
override a shop safety officer's judgment. Always-escalate paths
(human sign-off regardless of confidence, mapping this repo's Trust
Controls in [`docs/business-model.md`](docs/business-model.md)):
`:flag-safety-concern` (always) and `:coordinate-supply-order` above
the registered cost threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a print-shop scheduling/logistics coordination robot performs crew scheduling, job/materials-usage/progress-record logging and ink-paper-materials supply-order coordination for a print-shop crew, under an actor that proposes actions and an independent **Pressman Governor** that gates them. The governor never dispatches hardware itself, never operates the printing press itself, and never finalizes a press-run-execution decision or overrides a shop safety officer's judgment; `:high`/`:safety-critical` actions (such as a flagged machinery-hazard/ink-exposure concern, or an above-threshold supply order) require human sign-off. **This actor coordinates SHOP SCHEDULING/LOGISTICS ONLY — it never operates the printing press itself.**

## Core Contract

```text
crew roster + shop registration + safety-reporting policy
        |
        v
Pressman Advisor -> Pressman Governor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, finalize
a press-run-execution decision, override a shop safety officer's judgment,
suppress an operating record, or disclose sensitive data without governor
approval and audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `7322`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
