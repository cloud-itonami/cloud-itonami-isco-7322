(ns pressman.store
  "SSoT for the ISCO-08 7322 printers scheduling/logistics coordination
  actor (itonami actor pattern, ADR-2607121000 / CLAUDE.md Actors
  section; README's 'Robotics premise' — a print-shop scheduling/
  logistics coordination robot performs crew scheduling,
  job/materials-usage/progress-record logging and
  ink-paper-materials supply-order coordination for a print-shop
  crew under this advisor/governor pair, which never dispatches
  hardware itself, never operates a printing press itself, and never
  finalizes a press-run-execution decision or overrides a shop
  safety officer's judgment — those remain the shop safety officer's
  exclusive judgment). Modeled on cloud-itonami-isco-7223's
  machinist.store for the closely-related rotating-machinery
  safety-domain shape.

  Domain:

    worker — a registered print-shop crew member
             (:worker-id, :name)
    shop   — a registered print shop {:shop-id :name
             :max-supply-cost number}. `:max-supply-cost` is an
             informational registered ceiling used only to decide
             whether a `:coordinate-supply-order` proposal escalates
             to human sign-off (the governor never blocks a
             within-threshold order outright; it only decides
             commit vs. escalate).
    record — a committed operating record (a logged
             job/materials-usage/progress entry, a scheduled crew
             operation, a flagged safety concern, or a coordinated
             supply order) — written ONLY via commit-record!.
    ledger — append-only audit trail, commit or hold.")

(defprotocol Store
  (worker [s worker-id])
  (shop [s shop-id])
  (records-of [s worker-id])
  (ledger [s])
  (register-worker! [s worker])
  (register-shop! [s shop])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (worker [_ worker-id] (get-in @a [:workers worker-id]))
  (shop [_ shop-id] (get-in @a [:shops shop-id]))
  (records-of [_ worker-id] (filter #(= worker-id (:worker-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-worker! [s w]
    (swap! a assoc-in [:workers (:worker-id w)] w) s)
  (register-shop! [s sh]
    (swap! a assoc-in [:shops (:shop-id sh)] sh) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:workers {} :shops {} :records [] :ledger []}
                                    seed)))))
