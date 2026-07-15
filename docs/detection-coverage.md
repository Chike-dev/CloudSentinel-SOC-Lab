# CloudSentinel — Detection Coverage

This document summarizes what CloudSentinel currently detects, what was validated, and what remains planned.

---

## Validated coverage

| Behavior | Detection source | Automation path | Validation status |
|---|---|---|---|
| GuardDuty high-severity findings | GuardDuty sample findings | EventBridge → SNS + Lambda | Validated |
| EC2 role credential use outside AWS | GuardDuty real finding | EventBridge → SNS + Lambda | Validated |
| Finding aggregation | Security Hub | Dashboard visibility | Validated |
| Email alerting | SNS | Analyst notification | Validated |
| Incident parsing and logging | Lambda + CloudWatch Logs | Response evidence | Validated |

---

## Observed but not high-severity by itself

| Behavior | Observation | Interpretation |
|---|---|---|
| Internal AWS enumeration from EC2 | Commands such as `sts get-caller-identity`, S3 listing, IAM listing, role listing, and region listing were executed | Valid post-compromise reconnaissance, but not guaranteed to trigger a severity `>= 7` finding by itself |

This is a useful detection-engineering lesson: suspicious behavior does not always equal high-confidence detection. The automation threshold should reflect signal quality and operational impact.

---

## Planned coverage

| Behavior | Planned source | Notes |
|---|---|---|
| External port scanning | VPC Flow Logs + GuardDuty | Planned Simulation B |
| EC2 quarantine | Lambda + EC2 API | Future containment mode |
| Role-session revocation | IAM response workflow | Future credential containment improvement |
| Multi-region detection routing | EventBridge / Security Hub aggregation | Future production-hardening step |
| Third-party notification | Slack, Teams, PagerDuty, or ticketing | Future operational integration |

---

## Automation threshold

CloudSentinel currently routes findings where:

```json
{
  "detail": {
    "severity": [
      {
        "numeric": [">=", 7]
      }
    ]
  }
}
```

This threshold is intentionally conservative. High-severity findings trigger response automation; lower-severity findings remain available for manual review in GuardDuty and Security Hub.
