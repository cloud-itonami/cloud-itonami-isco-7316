(ns signcraft.store
  "SSoT for the ISCO-08 7316 sign & decorative-craft workshop
  scheduling/logistics coordination actor (itonami actor pattern,
  ADR-2607121000 / CLAUDE.md Actors section; README's 'Robotics
  premise' — a workshop scheduling/logistics coordination robot
  performs crew scheduling, job/commission/progress-record logging and
  paint/solvent/etching-materials supply-order coordination for a
  signwriting, decorative-painting, engraving and etching shop crew
  under this advisor/governor pair, which never dispatches hardware
  itself, never performs painting, engraving or etching work itself,
  and never finalizes a painting/engraving-execution decision or a
  workshop-chemical-safety-clearance decision, nor overrides a shop
  safety officer's judgment — those remain the shop safety officer's
  exclusive judgment). Modeled on cloud-itonami-isco-7321's
  prepresstech.store for the precision-craft/chemical-hazard-workshop
  shape.

  Domain:

    worker   — a registered signwriter, decorative painter, engraver
               or etcher shop crew member (:worker-id, :name)
    workshop — a registered sign & decorative-craft workshop
               {:workshop-id :name :max-supply-cost}.
               `:max-supply-cost` is an informational registered
               ceiling used only to decide whether a
               `:coordinate-supply-order` proposal escalates to human
               sign-off (the governor never blocks a within-threshold
               order outright; it only decides commit vs. escalate).
    record   — a committed operating record (a logged
               job/commission/progress entry, a scheduled crew
               operation, a flagged safety concern, or a coordinated
               supply order) — written ONLY via commit-record!.
    ledger   — append-only audit trail, commit or hold.")

(defprotocol Store
  (worker [s worker-id])
  (workshop [s workshop-id])
  (records-of [s worker-id])
  (ledger [s])
  (register-worker! [s worker])
  (register-workshop! [s workshop])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (worker [_ worker-id] (get-in @a [:workers worker-id]))
  (workshop [_ workshop-id] (get-in @a [:workshops workshop-id]))
  (records-of [_ worker-id] (filter #(= worker-id (:worker-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-worker! [s w]
    (swap! a assoc-in [:workers (:worker-id w)] w) s)
  (register-workshop! [s ws]
    (swap! a assoc-in [:workshops (:workshop-id ws)] ws) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:workers {} :workshops {} :records [] :ledger []}
                                    seed)))))
