# cloud-itonami-isco-7316

Open Occupation Blueprint for **ISCO-08 7316**: Signwriters, Decorative Painters, Engravers and Etchers.

This repository designs a forkable OSS business for a sign & decorative-craft workshop scheduling and logistics coordination practice: a workshop scheduling and supply-coordination robot manages crew/task records under a governor-gated actor, so a signwriting, decorative-painting, engraving and etching crew keeps its own operating records instead of renting a closed workforce-management SaaS.

**Maturity: `:implemented`.** `src/signcraft/` implements the
`SignCraftActor` as a `langgraph.graph/state-graph`
(`signcraft.actor`) wired to a `Sign & Decorative Craft Advisor`
(`signcraft.advisor`) and an independent `SignCraftGovernor`
(`signcraft.governor`), following the itonami actor pattern
(ADR-2607121000): `:intake -> :advise -> :govern -> :decide -+-> :commit
(:ok?) +-> :request-approval (:escalate?, human-in-the-loop interrupt)
+-> :hold (:hard?)`. 26 tests / 56 assertions green (`clojure -M:test`).
HARD invariants (always hold, never
overridable): worker provenance, workshop provenance, no-actuation
(`:effect` must be `:propose`), a closed op-allowlist
(`:log-work-record`, `:schedule-crew-operation`,
`:flag-safety-concern`, `:coordinate-supply-order` — nothing else may
ever be proposed), and a permanent, unconditional block on any
proposal that would directly finalize a painting/engraving-execution
decision (e.g. deciding to proceed with a specific signwriting,
decorative-painting, engraving or etching procedure) or a
workshop-chemical-safety-clearance decision, or override a shop
safety officer's judgment. Always-escalate paths (human sign-off
regardless of confidence, mapping this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)):
`:flag-safety-concern` (always) and `:coordinate-supply-order` above
the registered cost threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a workshop scheduling/logistics coordination robot performs crew scheduling, job/commission/progress-record logging and paint/solvent/etching-materials supply-order coordination for a sign & decorative-craft crew, under an actor that proposes actions and an independent **SignCraft Governor** that gates them. The governor never
dispatches hardware itself, never performs signwriting, decorative-painting, engraving or etching work on the shop floor, and never finalizes a painting/engraving-execution decision or a workshop-chemical-safety-clearance decision, nor overrides a shop safety officer's judgment; `:high`/`:safety-critical` actions (such as a flagged VOC/chemical-fume-exposure or equipment-condition concern, or an above-threshold supply order) require human sign-off. **This actor coordinates workshop scheduling/logistics only — it never performs signwriting, decorative-painting, engraving or etching work itself.**

## Core Contract

```text
crew roster + workshop registration + safety-reporting policy
        |
        v
Sign & Decorative Craft Advisor -> SignCraft Governor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, finalize
a painting/engraving-execution decision, declare a workshop-chemical-safety
clearance, override a shop safety officer's judgment, suppress an operating
record, or disclose sensitive data without governor approval and audit
evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `7316`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
