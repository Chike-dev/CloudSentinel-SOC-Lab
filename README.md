# CloudSentinel SOC Lab

> A cloud-native Security Operations Center (SOC) built entirely on AWS — real-time
> threat detection, automated incident response, and a **real** simulated attack that
> the pipeline detected on its own.

![AWS](https://img.shields.io/badge/AWS-Cloud-orange)
![GuardDuty](https://img.shields.io/badge/GuardDuty-Threat%20Detection-red)
![Lambda](https://img.shields.io/badge/Lambda-Python-blue)
![Security](https://img.shields.io/badge/Security-SOC-green)
![IaC](https://img.shields.io/badge/Terraform-Planned-lightgrey)

---

## What this is

CloudSentinel is a scaled-down but fully working SOC pipeline on AWS. It continuously
logs account activity, uses GuardDuty's ML-based detection to flag threats, aggregates
everything into a single dashboard, and automatically alerts and responds when a
high-severity finding appears — the same pattern real security teams run in production.

Then I attacked it. I stole an EC2 instance's credentials, used them from outside AWS,
and **GuardDuty caught it on its own** — no sample findings, a genuine detection that
flowed through the entire pipeline to an email alert in under half an hour.

---

## Architecture

![CloudSentinel Architecture](architecture/cloudsentinel-architecture.png)

**Data flow:** Account activity → CloudTrail (audit logging) → S3 + CloudWatch Logs
(storage) → GuardDuty (ML threat detection) → Security Hub (aggregation) → EventBridge
(routes findings with severity ≥ 7) → Lambda (automated response) **and** SNS (email
alert to the analyst).

---

## Technologies used

| Service | Role in the pipeline |
|---------|----------------------|
| **CloudTrail** | Records every API call — the complete audit trail |
| **S3 + CloudWatch Logs** | Centralized, encrypted log storage |
| **GuardDuty** | ML-based threat detection; produces severity-scored findings |
| **Security Hub** | Single-pane-of-glass aggregation of all findings |
| **EventBridge** | Event router; fires only on findings with severity ≥ 7 |
| **Lambda (Python)** | Serverless incident responder — parses, classifies, logs |
| **SNS** | Real-time email alerting to the SOC analyst |
| **IAM** | Least-privilege SOC analyst role (read-only) |

---

## How it was validated

The pipeline was proven in two stages — synthetic first, then real.

### 1. Pipeline validation (sample findings)

GuardDuty sample findings confirmed the full path works end to end:

| Stage | Evidence |
|-------|----------|
| GuardDuty raised findings | `screenshots/11-guardduty-sample-findings.png` |
| Security Hub aggregated them | `screenshots/12-SecurityHub-sample-findings.png` |
| SNS delivered email | `screenshots/13-sns-email-alert.png` |
| Lambda executed | `screenshots/14-lambda-cloudwatch-logs.png` |

*These are GuardDuty sample findings (`"sample": true`) — used to validate the wiring,
not claimed as a real attack.*

### 2. Real attack — Simulation A: credential exfiltration

I ran post-compromise reconnaissance inside a victim EC2 instance, pulled its IAM role
credentials from the instance metadata service (IMDSv2), and used them from an external
machine — a textbook credential-theft technique.

| Result | Value |
|--------|-------|
| **Finding** | `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS` |
| **Severity** | 8 / 10 (HIGH) |
| **Detection** | GuardDuty, unprompted, ~24 min after the attack |
| **Response** | EventBridge → SNS email delivered + Lambda incident logged |

Evidence: `screenshots/15-simulation-a-recon-commands.png`,
`screenshots/15b-credential-exfiltration-external.png`,
`screenshots/16-real-finding-guardduty.png`,
`screenshots/17-real-finding-email.png`,
`screenshots/18-securityhub-real-finding.png`

Full write-up: [`docs/attack-simulation-report.md`](docs/attack-simulation-report.md)

---

## Key concepts demonstrated

- **End-to-end threat detection pipeline** — from log generation to alert delivery.
- **Event-driven automation** — zero manual steps between detection and response.
- **Least-privilege IAM** — a scoped, read-only SOC analyst role.
- **Attacker + defender perspective** — built the detection *and* proved it works under
  a real attack.
- **Incident-response depth** — discovered that terminating a compromised instance does
  **not** invalidate already-stolen temporary credentials; they remain valid until
  expiry or explicit session revocation.

---

## Repository structure

```
CloudSentinel-SOC-Project/
├── README.md                     ← this page
├── architecture/                 ← architecture diagram
├── docs/
│   └── attack-simulation-report.md
├── eventbridge/
│   └── guardduty_high_severity_rule.json
├── lambda/
│   └── incident_responder.py
├── screenshots/                  ← full build + attack evidence (01–18)
└── terraform/                    ← Infrastructure as Code (planned, Phase 5)
```

---

## Roadmap

- **Simulation B** — external `nmap` port scan + VPC Flow Logs network-recon detection.
- **Terraform codification** — deploy the entire pipeline with `terraform apply` / tear
  down with `terraform destroy`.

---

*Built by [Chike-dev](https://github.com/Chike-dev) · AWS Free Tier · region us-east-1*
