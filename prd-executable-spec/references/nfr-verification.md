# Verifiable NFR Layer

Non-functional requirements (NFRs) are where systems that "work in the demo" die
in production. ISO/IEC/IEEE 29148 and ISO 25010 tell you *which* NFR dimensions
to cover; this layer tells you how to make each one **measurable and verifiable**
instead of leaving it as prose.

## Rule

Every NFR must have four parts. Prose like "the system should be fast" is a
`needs_clarification` issue, not an NFR.

| Part | Meaning | Example |
|------|---------|---------|
| `metric` | The thing being measured | request latency |
| `target` | The quantified threshold | p99 < 200ms |
| `condition` | The load / context the target holds under | @ 1000 rps, 1M rows |
| `verification_method` | How it will be proven | `load_test` |

## NFR object

```json
{
  "category": "performance",
  "metric": "api_latency_p99",
  "target": "< 200ms",
  "condition": "1000 rps sustained, 1M commission rows",
  "verification_method": "load_test"
}
```

`category` ∈ `performance | scalability | availability | security |
maintainability | compatibility | usability | portability | reliability`

`verification_method` ∈ `load_test | chaos_test | static_analysis | pen_test |
manual_review | monitoring_slo | benchmark`

## Examples by category

### Performance
```
metric: leaderboard_compute_duration
target: p99 < 60s
condition: up to 1,000,000 ranked users
verification_method: load_test
```

### Availability / Reliability
```
metric: settlement_job_success_rate
target: ≥ 99.9% monthly
condition: excluding scheduled maintenance windows
verification_method: monitoring_slo
```

### Scalability
```
metric: horizontal_throughput
target: linear to 8 workers (≥ 0.9 scaling efficiency)
condition: sharded by user_id
verification_method: load_test
```

### Security
```
metric: authz_enforcement
target: 0 endpoints accessible without a valid scope
condition: all /api/v1/* routes
verification_method: pen_test
```

## Turning an NFR into a runnable check

Where possible, express the NFR as an executable assertion so it can join CI or a
load-test suite:

```gherkin
@nfr @performance
Scenario: Commission query latency under load
  Given the commissions table has 1,000,000 rows
  When 1000 requests per second hit GET /api/v1/commissions for 5 minutes
  Then the p99 response time is below 200ms
  And the error rate is below 0.1%
```

Tag NFR scenarios (`@nfr`) so they can be run separately from functional
scenarios — they usually need a load harness, not a unit-test runner.
