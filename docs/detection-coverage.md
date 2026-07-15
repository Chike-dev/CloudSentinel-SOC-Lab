# CloudSentinel — Detection Coverage

This document summarizes what CloudSentinel currently detects and what was validated.

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
