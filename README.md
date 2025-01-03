# c7n-ec2-instance-state
This Cloud Custodian Policy takes EC2 instances that are created or stopped and sends the status to a Teams webhook.

There isn't a lot of documentation for using C7N to a Teams webhook. They want you to use the c7n-mailer but our environment has issues with it.
This policy is helpful to know if anyone in your organization has spun up a new EC2 without or permission or if one is terminated.
The main issue is knowing how to pull the correct information from the CloudWatch Events that are generated by this policy.
I plan to update this documentation as I discover the correct variables to pull from the event.

When this policy is generated, the event pulls from this message format:

{
  "version": "0",
  "id": "<id>",
  "detail-type": "EC2 Instance State-change Notification",
  "source": "aws.ec2",
  "account": "<account>",
  "time": "2024-12-30T14:11:47Z",
  "region": "us-east-1",
  "resources": [
    "<resourceARN>"
  ],
  "detail": {
    "instance-id": "<instanceid>",
    "state": "running"
  },
  "debug": true
}
