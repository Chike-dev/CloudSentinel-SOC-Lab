# CloudSentinel — Operational Runbook

This runbook describes how a SOC analyst should respond when CloudSentinel alerts on a high-severity GuardDuty finding.

---

## Alert source

CloudSentinel alerts are generated when EventBridge matches a GuardDuty finding with severity `>= 7`.

Response targets:

- SNS email notification to the analyst inbox
- Lambda incident responder logs in CloudWatch Logs
- Security Hub dashboard aggregation

---

## Initial triage checklist

When an alert arrives:

1. Open the SNS email and identify the finding type, severity, account, region, and timestamp.
2. Open GuardDuty and locate the finding.
3. Open Security Hub and confirm the finding is aggregated.
4. Open Lambda CloudWatch Logs and review the incident responder output.
5. Identify affected resources: EC2 instance, IAM role, IAM user, access key, source IP, or external account.
6. Establish an incident timeline using GuardDuty timestamps and CloudTrail events.

---

## Runbook: EC2 role credential exfiltration

Finding example:

```text
UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS
```

### Severity

High. This finding indicates that instance-profile credentials were observed outside the expected EC2 context.

### Triage questions

- Which IAM role was used?
- Which EC2 instance received the role credentials?
- Which external IP used the credentials?
- What API calls were made after the credentials were exfiltrated?
- Are the temporary credentials still valid?
- Did the attacker access S3, IAM, EC2, Secrets Manager, or other sensitive services?

### Containment considerations

Depending on impact and business context:

- Restrict or remove permissions from the compromised role.
- Revoke active sessions for the affected role.
- Isolate the EC2 instance using a quarantine security group.
- Detach the instance profile from the instance if appropriate.
- Preserve the instance for investigation before termination if forensic evidence is needed.
- Rotate any long-term credentials discovered during investigation.

### Investigation actions

- Review CloudTrail for API calls made by the compromised role.
- Search for actions after the first suspicious external credential use.
- Review S3 access, IAM enumeration, role assumption attempts, and data-access events if enabled.
- Compare source IPs against known corporate or expected networks.
- Identify whether credentials were used after the instance was terminated.

### Recovery actions

- Restore or rebuild affected workloads from known-good configuration.
- Reduce instance-role permissions to least privilege.
- Prefer IMDSv2 enforcement for EC2 workloads.
- Add monitoring for unusual role usage and external credential use.

---

## Lambda responder behavior in v1

The current Lambda responder runs in simulation mode. It:

- parses GuardDuty finding details,
- classifies severity,
- identifies affected EC2 resources if present,
- detects credential-related finding types,
- logs simulated containment actions to CloudWatch Logs.

It does not yet modify AWS resources.

---

## Production hardening candidates

Future versions of the responder could:

- create or apply a quarantine security group,
- detach the compromised instance profile,
- revoke active IAM role sessions,
- tag affected resources with incident metadata,
- open a ticket in a case-management system,
- notify Slack, Teams, or PagerDuty,
- and create a structured incident record in S3 or DynamoDB.
