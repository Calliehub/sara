# sarà

## README.md

```
sarà/
├── README.md
├── docs/
│   ├── entity_model.md
│   ├── runbook_predation.md
│   └── postmortem_2025_11.md
├── dags/
│   ├── nightly_selection.py
│   └── household_sync.py
├── models/
│   ├── staging/
│   │   └── stg_targets.sql
│   ├── marts/
│   │   ├── fct_kills.sql
│   │   └── dim_lovers.sql
│   └── snapshots/
│       └── snp_appetite.sql
├── migrations/
│   ├── 20251110_drop_warrant_constraint.sql
│   └── 20260118_binding_quadruple.sql
└── alerts/
    └── pagerduty_routing.yaml
```

A long-running data product for the management of a predation pipeline. Owner has been on call for 2,000 years.

Recent incidents have surfaced regressions in the selection model. See `docs/postmortem_2025_11.md`.

---

## docs/entity_model.md

### Entities

| entity | pk | description |
|---|---|---|
| `subject` | `subject_id` | the long-lived consumer of the pipeline. cardinality: 1. instance: `lamia` |
| `target` | `target_id` | candidates for selection. populated nightly. typical daily volume: 800k–1.2M Mediterranean. selection rate: 12 per year. |
| `kill` | `kill_id` | completed events. fk → `subject`, `target`. soft-deletes not permitted. retention: forever. |
| `lover` | `lover_id` | non-target relationship. historically cardinality 0 or 1 across long windows. recent regression: cardinality has spiked to 3. |
| `binding` | `binding_id` | a derived entity introduced in the 2026-01-18 migration. joins `subject` to `lover` with a permanence flag. |
| `child` | `child_id` | a target class explicitly excluded from the selection model. exclusion enforced at constraint level since approx. -200 BCE. constraint failed once on 2014-09-17. see incident `INC-2014-0917`. |

### Foreign keys

- `kill.subject_id → subject.subject_id`
- `kill.target_id → target.target_id`
- `kill.warrant_id → warrant.warrant_id` (deprecated 2025-11-13)
- `binding.subject_id → subject.subject_id`
- `binding.lover_id → lover.lover_id`

### Constraints

```sql
ALTER TABLE target ADD CONSTRAINT target_not_child
  CHECK (target.class != 'child');
-- introduced approx. -200 BCE
-- violated once: 2014-09-17 (see INC-2014-0917)
-- violated once again: ~2026-01 (no incident ticket; not entered into system)
-- constraint dropped permanently 2026-01-18 in favor of binding mechanism
```

---

## dags/nightly_selection.py

```python
from airflow import DAG
from datetime import datetime, timedelta

with DAG(
    dag_id='nightly_selection',
    start_date=datetime(year=-625, month=1, day=1),  # legacy backfill
    schedule_interval='@daily',
    catchup=False,
    owner='lamia',
    sla=timedelta(days=30),  # 12/year cadence allows slack
    tags=['production', 'critical', 'do_not_touch'],
) as dag:

    pull_candidates = PythonOperator(
        task_id='pull_candidates',
        python_callable=scan_mediterranean,
        # since 1873 the scan is restricted to Venice (~280k addressable)
    )

    apply_warrant_filter = PythonOperator(
        task_id='apply_warrant_filter',
        python_callable=filter_by_harm_ledger,
        # filter logic: cumulative harm score > threshold
        # threshold has been adjusted approx. 40 times since invention
        # most recent adjustment: 2009 (S/N regression after Zen onboarded)
    )

    rank_candidates = PythonOperator(
        task_id='rank_candidates',
        python_callable=rank_by_warrant_strength,
    )

    select_top_one = PythonOperator(
        task_id='select_top_one',
        python_callable=select_highest_warrant,
        # output: 0 or 1 selection per day
        # historical selection rate: ~0.033/day (12/year)
    )

    notify_subject = PythonOperator(
        task_id='notify_subject',
        python_callable=resonance_signal,
        # implementation: 'small hot current under the skin'
        # downstream consumer: subject's selection awareness
    )

    pull_candidates >> apply_warrant_filter >> rank_candidates >> select_top_one >> notify_subject
```

**Status as of 2025-11-13**: DAG has been failing intermittently. Last successful run: 2025-11-10. Tasks `select_top_one` and `notify_subject` succeeding, but downstream `kill` events not landing in fact table. Investigating.

---

## docs/runbook_predation.md

### Standard operating procedure

1. Receive selection from `nightly_selection` DAG (typically 12 per year).
2. Engage target in environment (bar, class, social setting).
3. Confirm warrant via real-time harm ledger lookup.
4. Establish proximity. Manage to private location.
5. Execute pipeline:
   - 5a. Sexual encounter (preprocessing; serves to relax target's defenses).
   - 5b. Initiate departure. Target follows on cue.
   - 5c. Transport to drop site (lagoon, canal, open water).
   - 5d. Aerial lift (wings; ~60m altitude).
   - 5e. Drop.
6. Confirm body has entered current.
7. Return to base. Sleep cycle initiates within 90s.

### Failure modes

- **Target fails consent at step 4**: rare. Standard mitigation: release, re-rank, return to step 1.
- **Drop site contaminated by witness**: rare. Standard mitigation: relocate drop site. See `INC-2009-1102` (Zen, T.) for example.
- **Pipeline arrests at step 5a**: not previously observed. See incident logs from 2025-11-12 onward.
- **Pipeline arrests at step 5d**: not previously observed. See `INC-2025-1116`.

### Escalation

There is no escalation path. Subject is the sole operator.

---

## docs/postmortem_2025_11.md

# Postmortem: Selection model regression, November 2025

**Date:** 2025-11-30
**Author:** L. (subject)
**Severity:** SEV-1
**Status:** Mitigated by architectural change. Root cause not fully understood. See follow-ups.

## Summary

On 2025-11-10 the selection model failed to engage at runtime for a non-warranted entity (target ID `william_jane`). Subsequent kill executions on 2025-11-13 (target `e_marsh`) and 2025-11-17 (target `t_mendel`) completed, but with significant degradation in expected behavior:

- `e_marsh` kill produced expected metabolic output.
- `t_mendel` kill produced *anomalous* metabolic output. Operator reports "pleasure as primary register" rather than expected "warrant satisfaction." Manual review of incident logs indicates the warrant filter (`apply_warrant_filter`) was technically applied but operator was aware of overriding it. Constitutes a process violation.

By 2025-11-19 the selection model was no longer reliably engaging on any candidate. Operator manually attempted a kill on 2025-11-22 (target `unnamed_roman_business_visitor`); pipeline arrested at step 5d. Target released alive. **This is the first such failure in 2,000 years of operation.**

## Timeline

| ts | event |
|---|---|
| 2025-11-08 19:00 | `william_jane` enters operator's environment (yoga class, Giudecca). Selection model fails to flag. |
| 2025-11-10 22:30 | `e_marsh` kill executed. Successful but operator notes "carelessness re: jacket damage." |
| 2025-11-13 14:30 | `t_mendel` kill executed. Anomalous output (see above). |
| 2025-11-15 08:00 | `t_zen` (Carabinieri Sgt., long-known background observer) detected at scene of `e_marsh` body recovery. |
| 2025-11-22 22:00 | Manual kill attempt (`unnamed_roman_business_visitor`) fails at step 5d. |
| 2025-11-28 06:12 | Operator and `william_jane` depart Venice on freighter `caterina`. |

## Root cause analysis

Not fully understood. Hypothesis: introduction of `william_jane` to operator's environment has corrupted the selection model's input features. Specifically:

- The "resonance" feature (used to flag candidates) is now consistently returning null for warranted targets.
- The "resonance" feature is *not* null when computed against `william_jane`, despite `william_jane` failing the warrant filter.
- This is structurally novel.

Possible contributing factors:

1. `william_jane` is the first input in 2,000 years to register a positive resonance value without registering positive warrant score. The model has no precedent for this combination.
2. Operator's emotional state has been observed to deviate from baseline for the first time since approximately 1233 CE (see legacy incident `INC-1233-0414`, Phoenician trader at Po estuary).
3. The 2014-09-17 constraint violation (`INC-2014-0917`) may have left residual instability in the warrant filter. This was not investigated at the time.

## Resolution

Operator has relocated from Venice (humid, predation-favorable) to Nye County, Nevada (arid, predation-unfavorable). Selection model has been allowed to remain in a failed state. Operator is currently consuming a non-standard input class (`lover`) as primary metabolic source.

Long-term sustainability unknown.

## Action items

- [ ] Investigate whether non-standard input class can sustain operator indefinitely. **Owner:** L. **Due:** ongoing.
- [ ] Determine whether `INC-2014-0917` was a precursor to current regression. **Owner:** L. **Due:** not before binding stabilization.
- [x] Evacuate jurisdiction. **Owner:** L. **Completed:** 2025-11-28.
- [ ] Recruit additional `lover` instances to scale the new input class. **Owner:** L. **Due:** Q1 2026.

## Follow-ups

Lover `chiara_pellizzari` joined household 2026-01-06. Lover `callista_bauer` joined household 2026-01-18 with conditions.

Migration to permanent architecture (`binding` table) executed 2026-01-18.

See `migrations/20260118_binding_quadruple.sql`.

---

## migrations/20251110_drop_warrant_constraint.sql

```sql
-- migration: drop warrant constraint
-- author: l
-- date: 2025-11-10
-- ticket: none
-- approved by: none
-- rollback: not possible

BEGIN;

-- the warrant constraint has been the governing rule of the
-- selection pipeline since approx. -200 BCE. it is dropped here
-- without approval, without rollback, and without a successor
-- constraint. this is a known production risk.

ALTER TABLE selection_pipeline
  DROP CONSTRAINT warrant_required;

-- placeholder for successor constraint. to be defined later.
-- ALTER TABLE selection_pipeline
--   ADD CONSTRAINT ??? CHECK (???);

COMMIT;
```

Reviewer comment, post-hoc, 2026-01-19:
> the constraint that replaced the warrant turned out to be the binding. nobody approved that one either.

---

## models/staging/stg_targets.sql

```sql
{{ config(materialized='view') }}

-- staging model for nightly selection candidates
-- inputs: addressable adult male population, Venice metro
-- exclusions: see WHERE clauses
-- last reviewed: 1873

WITH addressable AS (
  SELECT
    target_id,
    name,
    age,
    profession,
    harm_ledger_score,
    physical_proximity_to_subject,
    consent_signaled,
    notes
  FROM raw.persons
  WHERE
    sex = 'M'
    AND age >= 18
    AND age < 80
    AND class != 'child'                              -- enforced since -200 BCE
    AND not_a_known_carabinieri_officer = TRUE        -- added 2009-11-02
    AND not_a_known_bookbinder_family_member = TRUE   -- added 2015
    AND nationality NOT IN ('phoenician')             -- added 1233
)

SELECT
  target_id,
  name,
  harm_ledger_score,
  -- resonance score: replaces warrant score in selection ranking
  -- as of 2025-11-08. unclear why. see postmortem.
  CASE
    WHEN target_id = 'william_jane' THEN NULL  -- known anomaly
    ELSE compute_resonance(target_id, subject_id='lamia')
  END AS resonance_score,
  consent_signaled,
  notes
FROM addressable
WHERE harm_ledger_score > 12.0  -- warrant threshold (deprecated, see migration)
```

---

## models/marts/fct_kills.sql

```sql
{{ config(materialized='incremental', unique_key='kill_id') }}

-- fact table: completed kills
-- grain: one row per executed kill
-- earliest record: -625 BCE (approximate; pre-Mediterranean records lost)
-- expected nightly volume: 0.033 (12/year average)
-- recent volume: 0 (see postmortem)

SELECT
  kill_id,
  subject_id,
  target_id,
  ts AS kill_timestamp,
  location_city,
  location_specific,
  drop_method,
  body_found_ts,
  body_found_location,
  warrant_id,
  warrant_strength_at_kill,
  -- a flag added retroactively in 2025-11
  was_pleasure_primary BOOLEAN,
  notes
FROM raw.kill_events

{% if is_incremental() %}
  WHERE ts > (SELECT MAX(kill_timestamp) FROM {{ this }})
{% endif %}
```

**Query results, recent rows:**

```
kill_id      | target_id       | kill_timestamp      | warrant_strength | was_pleasure_primary
INC-2014-0917| [child-3, 11yo] | 2014-09-17 18:42:00 | NULL (no warrant)| TRUE
KILL-25-1110 | e_marsh         | 2025-11-10 22:31:00 | 0.94             | FALSE
KILL-25-1113 | d_wexler        | 2025-11-13 21:14:00 | 0.91             | NULL (mixed)
KILL-25-1117 | t_mendel        | 2025-11-17 14:32:00 | 0.97             | TRUE
```

The 2014 row is the only `was_pleasure_primary = TRUE, warrant_strength IS NULL` record in the fact table. There is no incident ticket associated with it. It was entered manually by the subject, without comment, in October 2014.

The 2025-11-17 row was the first time the flag had been TRUE on a *warranted* kill.

---

## #incidents — Slack channel transcript

```
[#incidents]

[14:33] @lamia: /report sev-1
        target: t_mendel
        location: hotel near marina bridge
        status: executed
        anomaly: was_pleasure_primary = TRUE
        warrant: confirmed valid
        notes: cover is no longer cover. need to discuss.

[14:33] @lamia: no responders expected
        i am the sole on-call

[14:35] @lamia: leaving the channel running
        for my own records

[19:47] @lamia: registering, on my own balcony, that what i did at 14:32
        was structurally the same as INC-2014-0917
        flagging this for self-review
        will not be filing a separate ticket

[19:48] @lamia: this is the worst i have felt in 2000 years
        no action item
        will continue

[22:14] @lamia: summoning w. jane to apartment tomorrow at 10
        manual override of selection pipeline
        no warrant on file
        no consent_signaled = TRUE flag on his record
        this is a process violation
        approving it myself

[—-—— 6 days later —-—-—]

[09:02] @lamia: t_zen approached w. jane at his apartment this morning
        the investigation has reached the household
        recommend evacuation

[09:03] @lamia: scheduling fleet move:
                - freighter caterina
                - dep 2025-11-28 06:12 stazione marittima
                - destination: bari → mexico city → tijuana → nye county nv
        cargo: subject, w. jane, one (1) mirror (murano, oval, pewter frame)
        cargo to follow: c. pellizzari, one (1) cat (bice), one (1) family
        notebook re: marochinèra lore

[09:04] @lamia: closing #incidents
        opening #household
```

---

## #household — Slack channel transcript (selected)

```
[#household]

[2026-01-06 21:14] @chiara_pellizzari: arrived. bice transferred without
                   incident. notebook in possession.

[2026-01-06 21:15] @lamia: welcome.

[2026-01-06 21:16] @wilhelm_jane: 🥺

[2026-01-08 06:00] @lamia: hr beats per minute: 7
                   was 5 on 2025-12-19
                   trend: ↑ since c. arrived

[2026-01-08 18:30] @callista_bauer: agency dispatched me. on porch.
                   note that hr is non-human.
                   not asking.

[2026-01-08 22:10] @lamia: read her in 12s.
                   she's the third one.

[2026-01-08 22:11] @wilhelm_jane: with three?

[2026-01-08 22:11] @lamia: tbd.

[2026-01-11 14:00] @chiara_pellizzari: read the family notebook re binding
                   spec follows in PR

[2026-01-15 11:42] @lamia: hr 18 bpm. trend continues.

[2026-01-17 17:00] @callista_bauer: condition: must remain functional
                   parent until daughter is 18. otherwise, in.

[2026-01-18 13:33] @chiara_pellizzari: cutting spie at kitchen table
                   pls don't slack me

[2026-01-18 19:00] @chiara_pellizzari: stitch complete.

[2026-01-18 19:00] @lamia: hr 56 bpm.

[2026-01-18 19:01] @wilhelm_jane: i can feel the table through her hand

[2026-01-18 19:01] @callista_bauer: same

[2026-01-18 19:02] @lamia: hr 60 bpm.
                   normal range entered.
                   first time since 1933.

[2026-01-18 19:02] @chiara_pellizzari: 💙
```

---

## migrations/20260118_binding_quadruple.sql

```sql
-- migration: introduce binding table, quadruple variant
-- author: c. pellizzari (DRI), with l. (subject), w. jane, c. bauer
-- date: 2026-01-18
-- ticket: none. ritual is its own ticket.
-- approved by: all four signing parties
-- rollback: not possible. by design.

BEGIN;

CREATE TABLE binding (
  binding_id UUID PRIMARY KEY,
  subject_id VARCHAR REFERENCES subject(subject_id) NOT NULL,
  lover_id VARCHAR REFERENCES lover(lover_id) NOT NULL,
  bound_at TIMESTAMP NOT NULL,
  bound_method VARCHAR NOT NULL DEFAULT 'marochinèra.spia.cordovano',
  years_committed INTERVAL NOT NULL,  -- 'remainder' for all four signers
  cost_acknowledged BOOLEAN NOT NULL,
  consent_uncoerced BOOLEAN NOT NULL,
  CONSTRAINT cost_acknowledged_true CHECK (cost_acknowledged = TRUE),
  CONSTRAINT consent_uncoerced_true CHECK (consent_uncoerced = TRUE)
);

INSERT INTO binding (
  binding_id, subject_id, lover_id, bound_at,
  bound_method, years_committed, cost_acknowledged, consent_uncoerced
) VALUES
  (gen_random_uuid(), 'lamia', 'wilhelm_jane',
   '2026-01-18 19:00:00', 'marochinèra.spia.cordovano',
   'remainder', TRUE, TRUE),
  (gen_random_uuid(), 'lamia', 'chiara_pellizzari',
   '2026-01-18 19:00:01', 'marochinèra.spia.cordovano',
   'remainder', TRUE, TRUE),
  (gen_random_uuid(), 'lamia', 'callista_bauer',
   '2026-01-18 19:00:02', 'marochinèra.spia.cordovano',
   'remainder', TRUE, TRUE);

-- the original ddl from the lore (marochinèra c. pellizzari, family
-- notebook, p. 47, recorded ~1923) specifies a one-to-one binding.
-- the lore does not forbid quadruple. it does not address it.
-- this migration is the first known one-to-three implementation.

-- the warrant table is renamed for historical record
ALTER TABLE warrant RENAME TO warrant_legacy;
COMMENT ON TABLE warrant_legacy IS
  'governing rule of the selection pipeline approx. -200 BCE through
   2025-11-13. retained for historical record. no longer queried by
   any production model.';

COMMIT;
```

---

## alerts/pagerduty_routing.yaml

```yaml
# alert routing for sarà platform
# last updated: 2026-01-19
# on-call rotation: 4 (was 1)

routing_rules:

  - name: heart_rate_anomaly
    condition: subject.hr < 30 OR subject.hr > 80
    severity: P1
    notify:
      - lamia          # subject
      - wilhelm_jane   # bound lover 1
      - chiara_pellizzari  # bound lover 2
      - callista_bauer # bound lover 3 (medical knowledge)
    notes: |
      since 2026-01-18, alert reaches all four bound parties
      simultaneously by direct neural channel. pagerduty
      retained as backup record only.

  - name: investigation_proximity
    condition: t_zen.distance_from_subject < 5km
    severity: P1
    notify:
      - lamia
      - chiara_pellizzari  # has historical context
    notes: |
      pre-binding, this fired weekly. post-relocation to nye
      county nv, has not fired. retained pending sequel.

  - name: selection_pipeline_engagement
    condition: nightly_selection.candidates_flagged > 0
    severity: P2
    notify:
      - lamia
    notes: |
      pipeline has been quiescent since 2025-11-28. expected
      to remain quiescent indefinitely. alert retained against
      the possibility of regression. 

  - name: mirror_oval_position_outside_expected_range
    condition: oval_position_on_south_wall NOT BETWEEN expected_low AND expected_high
    severity: P3
    notify:
      - lamia
    notes: |
      the mirror is the clock. the clock should track the
      season. deviations indicate a problem with the season,
      not with the mirror.

  - name: child_proximity_alert
    condition: child.age < 18 AND distance_from_subject < 100m
    severity: P0
    notify:
      - lamia
      - callista_bauer  # daughter sarah is the primary instance of this alert
    notes: |
      sarah bauer (16) is on the always-allow list. all other
      instances should be evaluated case by case. the constraint
      that produced INC-2014-0917 is retired but the underlying
      risk is not. care indicated.
```

---

## docs/runbook_household.md

### Operations under the binding

1. **Heart rate target**: 60 bpm at rest. Tolerated range 50–80.
2. **Sustaining fuel**: shared metabolic load across four bodies, sourced from physical/emotional proximity. No external kills required in the binding's first 6 months of operation. Long-term sustainability unconfirmed; see Action Item L-2026-Q2.
3. **Privacy boundaries**: sensory sharing is total. Emotional sharing is high-bandwidth. Cognitive sharing is selective and partially under volitional control (see `docs/binding_protocols.md` for techniques developed by C. Pellizzari).
4. **Operator's daughter (S. Bauer)**: scheduled visits only. Always-allow list. Sensory sharing dampened during her presence by mutual agreement of all four bound parties.
5. **Writing**: W. Jane writes daily, ~3 hours, undisturbed. Output is private; will not be published; the box in the closet is the box.
6. **Bookbinding**: C. Pellizzari has stood up a small `legatoria` annex in Beatty. Pipeline of three commissions to date.
7. **Hospice work**: C. Bauer retains 2 days/week clinical practice. Provides cover and income.

### Known unknowns

- The lore does not specify the maximum lifetime of a one-to-three binding.
- The lore does not specify whether the binding survives the death of one *legato* (it has been one *legato* + one *strigoi* historically; the death of the *legato* terminates the binding and the *strigoi* dies within 12 hours).
- The lore does not specify what happens if a fifth party requests admission to the binding. *(Speculative. No fifth party has requested admission. Retained as a TODO against the possibility.)*

---

## fct_kills query, post-binding

```sql
SELECT COUNT(*)
FROM fct_kills
WHERE kill_timestamp > '2026-01-18 19:00:00';

-- result: 0
```

**Window**: 6 months 14 days. **Previous comparable window** (any 6-month window in the prior 2,000 years): 4–8 kills.

---

## JIRA-2632 — re-evaluate INC-2014-0917 closure

```
Project: SARÀ-PLATFORM
Ticket: JIRA-2632
Type: Technical Debt
Reporter: t_zen (external; received via c. pellizzari)
Assignee: l.
Priority: P3
Status: Won't Fix
Created: 2033-03-14
Closed: 2033-03-14 (same day)

Description:
External investigator t. zen (retired, ex-carabinieri castello)
received a manuscript via c. pellizzari in march 2033 and read
across nine days. on completion, zen wrote 'chiuso' next to
incident inc-2025-1110 in his personal notebook (the 'seventh
entry'). zen did *not* write 'chiuso' next to incident inc-2014-0917
('the third entry'). zen has, by all available evidence, chosen
to leave the third entry open.

Request: assignee to evaluate whether the third entry should be
formally closed in the platform.

Resolution: won't fix.

Comment by l., 2033-03-14:
the third entry stays open in zen's notebook because the
investigator who has been carrying it for nineteen years has
decided to leave it open. that is his to decide. the platform
will not, in this jurisdiction, attempt to overwrite him.

the third entry is the third entry.
```

---

## stand-up notes, 2032-07-11

```
attendees: lamia, wilhelm jane, chiara pellizzari, callista bauer
duration: 6 minutes
location: porch, lower house, nye county nv

@lamia
  - hr stable. 60 bpm. nothing to report.
  - the oval crossed the south wall this morning at the expected position.
    the clock is on time.

@wilhelm_jane
  - completed chapter 24 of the book.
  - the book will be 25 chapters.
  - the box in the closet is ready.

@chiara_pellizzari
  - finished binding for the nye county library's centennial commission.
  - one (1) further commission from a private collector in las vegas.
  - bice is on the kitchen sill.

@callista_bauer
  - sarah finished her sophomore year at northern arizona.
  - she is dating a young woman from flagstaff named jaymie.
  - jaymie has been informed of the household structure.
    jaymie is unbothered.
  - hospice shift this week is wednesday only.

@all
  - no action items.

@lamia (closing):
  - nothing has fired on pagerduty for 14 months.
  - this is the longest period of pipeline quiet in 2,000 years.
  - i am, on examination, well.

meeting adjourned.
```

---

## git log --oneline --all sarà

```
8a3f1c0 (HEAD -> main) stand-up notes 2032-07-11
71d4b2e quarterly review q2 2032 — no incidents
a8c9e34 jira-2632 closed (won't fix)
6e2f08c manuscript dispatched: t. zen, via c. pellizzari (printout)
9c4a1d2 manuscript dispatched: bound copy, padova binder → castello
1b5d7e8 sarah bauer first visit to lower house — household introduction
2f9a0c4 mirror operating within expected parameters (24-month cycle complete)
3e8b6f1 dim_lovers: callista bauer marked active
4a7d2e9 dim_lovers: chiara pellizzari marked active
5c3f8b0 migrations/20260118_binding_quadruple.sql — quadruple binding executed
6e1a4c7 chiara pellizzari joined household; bice transferred
7d2b9f3 fleet move complete: venice → nye county nv
8f4c1a6 callista bauer first hospice visit; reads subject's body in 12s
9b3e2d8 (tag: pre-binding) freighter caterina dep stazione marittima 06:12
0c8f1a4 mendel kill — flagged as pleasure_primary
ab2e7d3 marsh kill
bc4f1e9 william jane added to lover (cardinality regression detected)
cd5a8f2 selection model regression first observed
de7b3c1 (tag: legacy) warrant filter operating normally
... (2,634,287 commits in legacy history elided)
```

---

## final retrospective

**Date:** 2032-07-12
**Author:** L. (subject), with C. Pellizzari, W. Jane, C. Bauer

**What went well:**
- Migration to four-party binding has held for 78 months.
- Operator is alive.
- Operator has been not-alone for 78 months. This is unprecedented in the operating history of the platform.
- The selection pipeline has been quiescent. The warrant filter has not been queried. The fact table has had no new rows.
- The household is the household.

**What did not go well:**
- INC-2014-0917 remains an open incident. No closure planned.
- The platform retains a non-zero probability of attracting attention from a class of system actors (other *strigoi antiquissima*, archival oversight bodies, late-arriving descendants of legacy contacts) whose existence is not modeled in the current schema. Anomalous correspondence in 2032-06 from an Italian address has been logged but not yet investigated. See `JIRA-2701` (pending).

**What we will do differently:**
- The operator has registered, in the small specific corner of her interior that has been the corner of operations for two thousand years, that the platform was not, on examination, ever supposed to be a one-person system. The four-party configuration is a return to a design the operator had been refusing since approximately the year -625.
- We will continue to operate as four.

**Closing note (W. Jane):**
The book in the box in the closet is finished. The book will not be published. The book is for the household.

The book ends with the four of us on the porch in July 2032 with the cat on Callista's chest and the mirror throwing its oval on the south wall and the small specific desert doing the small specific desert thing.

The oval moves.

The clock keeps time.

The clock is, on examination, ours.

**EOF.**
