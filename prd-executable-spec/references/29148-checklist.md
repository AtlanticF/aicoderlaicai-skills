# ISO/IEC/IEEE 29148 Completeness Checklist

Use this checklist in Phase 1 to scan the PRD for missing dimensions.

```
□ Purpose & Scope
□ Terms & Definitions
□ Stakeholder Requirements
  □ User characteristics  □ Operational scenarios  □ Constraints
□ System Requirements
  □ Functional Requirements
  □ External Interface Requirements
    □ User interface  □ Hardware interface  □ Software interface  □ Communication interface
  □ Performance Requirements
    □ Response time  □ Throughput  □ Concurrency limits  □ Resource usage caps
  □ Operational Requirements
    □ Timezone / locale handling  □ Scheduled task trigger rules  □ Failure recovery strategy
  □ Security Requirements
  □ Maintainability Requirements
□ Design Constraints
□ Quality Characteristics (ISO 25010)
  □ Functional suitability  □ Performance efficiency  □ Compatibility
  □ Usability  □ Reliability  □ Security  □ Maintainability  □ Portability
□ Verification Requirements
□ Assumptions & Dependencies
□ Traceability — requirements traceable to source
```

> **Key additions over IEEE 830**: Stakeholder Requirements (align on who needs what before building), Operational Requirements (timezone, scheduled trigger rules, runtime behavior), Verification Requirements (how each requirement is validated), and traceability.

> **Make NFRs verifiable, not prose**: Any item under Performance, Security,
> Operational, or Quality Characteristics that is flagged ❌ or is stated only in
> prose must be translated into a measurable NFR (metric + target + condition +
> verification method) via `references/nfr-verification.md`. A performance/quality
> requirement without a number is a `needs_clarification` issue, not a spec.
