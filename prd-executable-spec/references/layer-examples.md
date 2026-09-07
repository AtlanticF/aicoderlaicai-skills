# Layer Examples

Annotated examples for each layer of the five-layer precision model.

## BDD Gherkin (Meso — Behavior Layer)

> **Why this example matters**: The PRD said "calculate once per day." AI inferred UTC 00:00. The actual requirement had three hidden constraints: local timezone, post-scoring-task completion, and a hard deadline of 02:00. Gherkin makes all three explicit.

```gherkin
Feature: M03-F01 Leaderboard Calculation
  Background:
    Given the system is operating normally

  Scenario: M03-F01-R01 Daily leaderboard calculation — triggered after scoring task
    Given the scoring task for the current day has completed
    And the current time in the user's local timezone is before 02:00
    When the system triggers leaderboard calculation
    Then the leaderboard is computed based on the completed scoring results
    And the calculation completes before 02:00 local time

  Scenario: M03-F01-R01 Daily leaderboard calculation — scoring task not yet complete
    Given the scoring task for the current day has NOT completed
    When 02:00 local time is reached
    Then the system does NOT trigger leaderboard calculation
    And an alert is raised for manual review

  Scenario: M03-F01-R02 Leaderboard calculation — timezone handling
    Given users are distributed across multiple timezones
    When leaderboard calculation is triggered
    Then the trigger time is evaluated in each user's local timezone
    And NOT in UTC+0
```

---

## DMN Decision Tables (Meso — Decision Layer)

Use DMN when:
- Multiple input conditions map to a discrete output (e.g., tier → rate, role → permissions)
- A triggered action depends on a combination of states
- The rule table would otherwise be expressed as a nested if/else chain

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="https://www.omg.org/spec/DMN/20191111/MODEL/"
             xmlns:dmndi="https://www.omg.org/spec/DMN/20191111/DMNDI/"
             id="leaderboard-trigger" name="Leaderboard Trigger Rules" namespace="http://example.com">

  <decision id="shouldTriggerLeaderboard" name="Should Trigger Leaderboard">
    <decisionTable id="leaderboardTable" hitPolicy="FIRST">
      <input id="scoringTaskDone" label="Scoring task completed">
        <inputExpression typeRef="boolean"/>
      </input>
      <input id="localTimeBefore0200" label="Local time before 02:00">
        <inputExpression typeRef="boolean"/>
      </input>
      <output id="trigger" label="Trigger leaderboard" typeRef="boolean"/>
      <output id="reason" label="Reason" typeRef="string"/>
      <rule>
        <inputEntry><text>true</text></inputEntry>
        <inputEntry><text>true</text></inputEntry>
        <outputEntry><text>true</text></outputEntry>
        <outputEntry><text>"Scoring done, within time window"</text></outputEntry>
      </rule>
      <rule>
        <inputEntry><text>false</text></inputEntry>
        <inputEntry><text>-</text></inputEntry>
        <outputEntry><text>false</text></outputEntry>
        <outputEntry><text>"Scoring task not yet complete"</text></outputEntry>
      </rule>
      <rule>
        <inputEntry><text>-</text></inputEntry>
        <inputEntry><text>false</text></inputEntry>
        <outputEntry><text>false</text></outputEntry>
        <outputEntry><text>"Past 02:00 deadline"</text></outputEntry>
      </rule>
    </decisionTable>
  </decision>

</definitions>
```

Save each decision as a separate `.dmn` file: `{module}-{decision-name}.dmn`

---

## Pseudocode + Math (Micro — Logic Layer)

### Computation logic

```
FUNCTION calculate_commission(trade):
  base = trade.volume × trade.fee_rate

  rate = CASE user.tier OF
    "bronze"  → 0.10
    "silver"  → 0.20
    "gold"    → 0.30
    DEFAULT   → ERROR("unknown tier: " + user.tier)
  END CASE

  commission = base × rate
  commission = ROUND_DOWN(commission, 8)  // truncate, do not round

  ASSERT commission ≥ 0
  RETURN commission
```

### Math formula

```
commission_i = ROUND_DOWN(volume_i × fee_rate_i × tier_rate(user), 8)

total_commission = Σ commission_i,  i ∈ trades(user, period)

WHERE:
  tier_rate: {bronze: 0.10, silver: 0.20, gold: 0.30}
  period = [T_start, T_end), T_end - T_start = 24h
  fee_rate_i ∈ (0, 1]
```

### State machine

```
STATE_MACHINE: OrderSettlement
  STATES: [pending, confirmed, settling, settled, failed]

  TRANSITIONS:
    pending    → confirmed  : WHEN payment_verified = true
    pending    → failed     : WHEN timeout > 3600s OR payment_rejected
    confirmed  → settling   : WHEN T_current ≥ T_trade + T+1
    settling   → settled    : WHEN ledger_committed = true
    settling   → failed     : WHEN retry_count > 3
    failed     → pending    : WHEN manual_retry BY admin

  INVARIANTS:
    - settled is a terminal state, irreversible
    - failed → pending only allowed via admin action
```

### Common math patterns

```
// Summation
total = Σ(item_i × rate_i),  i ∈ [1, n]

// Conditional branching
result = CASE
  WHEN x > threshold → A
  WHEN x = threshold → B
  OTHERWISE          → C

// Time sequence
T_end = T_start + Δt_process + Δt_confirm
  WHERE Δt_confirm ≥ 0

// Proportional allocation
share_a = total × (weight_a / Σ weight_i)

// Precision
value = ROUND_DOWN(raw_value, decimals)
```

---

## JSON Schema / OpenAPI (Foundation — Data Layer)

### Data entity

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "CommissionRecord",
  "type": "object",
  "required": ["id", "user_id", "trade_id", "amount", "status", "created_at"],
  "properties": {
    "id":         { "type": "integer", "minimum": 1 },
    "user_id":    { "type": "integer", "minimum": 1 },
    "trade_id":   { "type": "string", "pattern": "^[A-Z0-9]{16,32}$" },
    "amount":     { "type": "string", "pattern": "^[0-9]+(\\.[0-9]{1,8})?$", "description": "Stored as string to avoid floating point precision loss" },
    "currency":   { "type": "string", "enum": ["USDT", "BTC", "ETH"] },
    "status":     { "type": "string", "enum": ["pending", "frozen", "released", "cancelled"] },
    "created_at": { "type": "string", "format": "date-time" }
  }
}
```

### Interface definition (OpenAPI fragment)

```yaml
/api/v1/commissions:
  get:
    summary: Query user commission records
    parameters:
      - name: user_id
        in: query
        required: true
        schema: { type: integer, minimum: 1 }
      - name: start_date
        in: query
        required: true
        schema: { type: string, format: date }
      - name: end_date
        in: query
        required: true
        schema: { type: string, format: date }
      - name: page
        in: query
        schema: { type: integer, minimum: 1, default: 1 }
      - name: page_size
        in: query
        schema: { type: integer, minimum: 1, maximum: 100, default: 20 }
    responses:
      '200':
        description: Success
        content:
          application/json:
            schema:
              type: object
              properties:
                total: { type: integer }
                items:
                  type: array
                  items: { $ref: '#/components/schemas/CommissionRecord' }
      '400':
        description: Invalid parameters
      '401':
        description: Unauthorized
```

---

## Entity Impact (Foundation — Data Layer, grounded in the real system)

> **Why this matters**: A JSON Schema alone reads as greenfield. Engineers must
> know whether a field lands in a new table or mutates a hot existing one — that
> single fact can swing an estimate from hours to a migration project.

```json
{
  "new_entities": ["LeaderboardSnapshot"],
  "modified_entities": ["ScoringResult"],
  "migration_notes": "Add nullable column ScoringResult.completed_at (backfill required); add index (completed_at). No downtime; backfill in batches of 10k."
}
```

Checklist:
- New entity → provide the full JSON Schema.
- Modified entity → list only the changed/added fields and the migration impact
  (nullable? backfill? index? lock risk on a large table?).

---

## Error / Idempotency / Observability Contract

> **Why this matters**: Failure paths and on-call debugging are where interfaces
> actually break down. Make the contract explicit, not implied by a "sad path"
> scenario.

### Error contract

```json
{
  "errors": [
    { "code": "SCORING_NOT_READY", "http_status": 409, "when": "leaderboard triggered before scoring completes", "retriable": true },
    { "code": "INVALID_TIMEZONE",  "http_status": 400, "when": "user timezone is unknown/unset",              "retriable": false }
  ],
  "idempotent": true,
  "idempotency_key": "date + timezone",
  "retry_policy": "exponential backoff, max 5 attempts, then alert on-call"
}
```

Rules:
- Every failure-path Gherkin scenario maps to one `errors[]` entry.
- If `idempotent: true`, define the `idempotency_key`.
- Every `retriable: true` error has a `retry_policy`.

### Observability

```json
{
  "logs": ["leaderboard.trigger.received", "leaderboard.compute.completed", "leaderboard.compute.failed"],
  "metrics": ["leaderboard_compute_duration_ms (histogram)", "leaderboard_compute_failures_total (counter)"],
  "alerts": ["scoring not complete by 02:00 local time", "compute p99 > 60s"]
}
```

---

## Test-Stub Generation (making the spec runnable)

Each `.feature` can be turned into failing step stubs so the spec becomes the
starting point of the test suite. Example (pytest-bdd):

```python
# m03-leaderboard.steps.py  (generated from m03-leaderboard.feature)
from pytest_bdd import scenarios, given, when, then

scenarios("m03-leaderboard.feature")

@given("the scoring task for the current day has completed")
def scoring_completed():
    raise NotImplementedError  # TODO: implement

@when("the system triggers leaderboard calculation")
def trigger_calculation():
    raise NotImplementedError  # TODO: implement

@then("the leaderboard is computed based on the completed scoring results")
def assert_leaderboard_computed():
    raise NotImplementedError  # TODO: implement
```

The stubs fail until implemented, so "spec done" and "tests defined" become the
same milestone. Emit one `.steps.<ext>` per `.feature`; pick the ecosystem's BDD
runner (pytest-bdd, Cucumber, behave, godog, etc.).
