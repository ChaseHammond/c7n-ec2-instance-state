# c7n-ec2-instance-state
This Cloud Custodian Policy takes EC2 instances that are created or stopped and sends the status to a Teams webhook.

Look here for setting up a Teams webhook: https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook?tabs=newteams%2Cdotnet

There isn't a lot of documentation for using C7N to a Teams webhook. They want you to use the c7n-mailer but our environment has issues with it.
This policy is helpful to know if anyone in your organization has spun up a new EC2 without permission or if one has been terminated.
The main issue is knowing how to pull the correct information from the CloudWatch Events that are generated by this policy.

This policy creates a CloudWatch event. You can then use a JMESPath query to pull the information you want and send it to your webhook.
