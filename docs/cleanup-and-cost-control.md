# CloudSentinel — Cleanup and Cost Control

This project uses AWS services that may incur charges if left running. Cleanup is part of the engineering workflow.

---

## Immediate cleanup after validation

### EC2

- Terminate the `cloudsentinel-victim` instance after simulations are complete.
- Delete unused EBS volumes if any remain.
- Delete the temporary security group if it is no longer attached.
- Keep the `.pem` key private and never commit it to GitHub.

### IAM

- Review temporary lab roles such as `CloudSentinel-EC2ReconRole`.
- Remove roles that are no longer needed.
- Keep the read-only `CloudSentinel-SOCAnalyst` role only if you want to preserve the access-design evidence.

### GuardDuty and Security Hub

- Disable GuardDuty after the project if you do not want ongoing analysis charges.
- Disable Security Hub after the project if you do not want ongoing findings or posture-management charges.

### CloudTrail, S3, and CloudWatch Logs

- Delete the CloudTrail trail if the lab is complete.
- Review the S3 log bucket and delete it if you do not need long-term evidence.
- Review CloudWatch log groups and set retention or delete them.

### SNS, EventBridge, and Lambda

- These are usually low-cost at lab scale, but can be deleted if the project is complete.
- Delete EventBridge rules if you disable GuardDuty and no longer need routing.
- Delete Lambda functions if you do not plan to reuse them.

---

## Recommended cost controls

- Create an AWS Budget with email alerts.
- Review Billing and Cost Management after enabling GuardDuty and Security Hub.
- Use one small EC2 instance only during testing.
- Terminate compute resources immediately after simulations.
- Avoid enabling data events or high-volume logging unless required.
- Set CloudWatch Logs retention periods instead of keeping logs indefinitely.

---

## Cleanup checklist

```text
[ ] Terminate EC2 victim instance
[ ] Delete unattached EBS volumes
[ ] Delete temporary EC2 security group
[ ] Remove temporary EC2 reconnaissance IAM role
[ ] Review or delete CloudTrail trail
[ ] Review or delete S3 log bucket
[ ] Set CloudWatch log retention or delete log groups
[ ] Disable GuardDuty if no longer testing
[ ] Disable Security Hub if no longer testing
[ ] Delete EventBridge rule if no longer needed
[ ] Delete Lambda function if no longer needed
[ ] Delete SNS topic/subscription if no longer needed
[ ] Confirm current month bill remains expected
```
