# c7n-ec2-instance-state
This Cloud Custodian Policy takes EC2 instances running or stopped and sends the status to a Teams webhook.

Look here for setting up a Teams webhook: https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook?tabs=newteams%2Cdotnet

There isn't a lot of documentation on using C7N for a Teams webhook. You can use the c7n-mailer but our environment has issues.
This policy is helpful if anyone in your organization has spun up a new EC2 without permission or if one has been stopped.
The main issue is knowing how to pull the correct information from the CloudWatch Events that are generated.

This policy creates a CloudWatch event for the EC2 instance state change. You can then use a JMESPath query to pull the information you want and send it to your webhook. When you look at the logs for the event you can see the message which looks something like this: 

<REQUEST-ID>  Processing event {     
"version": "0",     
"id": "<EVENT-ID>",     
"detail-type": "EC2 Instance State-change Notification",     
"source": "aws.ec2",     
"account": "<ACCOUNT-ID>",     
"time": "2024-12-30T20:07:06Z",     
"region": "us-east-1",     
"resources": [         
"arn:aws:ec2:us-east-1:<ACCOUNT-ID>:instance/<INSTANCE-ID>"     
],     
"detail": {         
"instance-id": "<INSTANCE-ID>",         
"state": "stopping"     
},     
"debug": true 
}   

[INFO]  2024-12-30T20:07:08.588Z  <REQUEST-ID>  Resources [{'AmiLaunchIndex': 0, 'ImageId': '<AMI-ID>', 'InstanceId': '<INSTANCE-ID>', 'InstanceType': 't2.micro', 'LaunchTime': datetime.datetime(2024, 12, 30, 18, 49, 40, tzinfo=tzlocal()), 'Monitoring': {'State': 'disabled'}, 'Placement': {'AvailabilityZone': 'us-east-1b', 'GroupName': '', 'Tenancy': 'default'}, 'PrivateDnsName': '<PRIVATE-DNS>', 'PrivateIpAddress': '<PRIVATE-IP>', 'ProductCodes': [], 'PublicDnsName': '', 'State': {'Code': 64, 'Name': 'stopping'}, 'StateTransitionReason': 'User initiated (2024-12-30 20:07:06 GMT)', 'SubnetId': '<SUBNET-ID>', 'VpcId': '<VPC-ID>', 'Architecture': 'x86_64', 'BlockDeviceMappings': [{'DeviceName': '/dev/xvda', 'Ebs': {'AttachTime': datetime.datetime(2024, 12, 19, 19, 58, 42, tzinfo=tzlocal()), 'DeleteOnTermination': True, 'Status': 'attached', 'VolumeId': '<VOLUME-ID>'}}], 'ClientToken': '<CLIENT-TOKEN>', 'EbsOptimized': False, 'EnaSupport': True, 'Hypervisor': 'xen', 'IamInstanceProfile': {'Arn': 'arn:aws:iam::<ACCOUNT-ID>:instance-profile/<PROFILE-NAME>', 'Id': '<PROFILE-ID>'}, 'NetworkInterfaces': [{'Attachment': {'AttachTime': datetime.datetime(2024, 12, 19, 19, 58, 41, tzinfo=tzlocal()), 'AttachmentId': '<ATTACHMENT-ID>', 'DeleteOnTermination': True, 'DeviceIndex': 0, 'Status': 'attached', 'NetworkCardIndex': 0}, 'Description': '', 'Groups': [{'GroupName': 'default', 'GroupId': '<SECURITY-GROUP-ID>'}], 'Ipv6Addresses': [], 'MacAddress': '<MAC-ADDRESS>', 'NetworkInterfaceId': '<INTERFACE-ID>', 'OwnerId': '<ACCOUNT-ID>', 'PrivateDnsName': '<PRIVATE-DNS>', 'PrivateIpAddress': '<PRIVATE-IP>', 'PrivateIpAddresses': [{'Primary': True, 'PrivateDnsName': '<PRIVATE-DNS>', 'PrivateIpAddress': '<PRIVATE-IP>'}], 'SourceDestCheck': True, 'Status': 'in-use', 'SubnetId': '<SUBNET-ID>', 'VpcId': '<VPC-ID>', 'InterfaceType': 'interface'}], 'RootDeviceName': '/dev/xvda', 'RootDeviceType': 'ebs', 'SecurityGroups': [{'GroupName': 'default', 'GroupId': '<SECURITY-GROUP-ID>'}], 'SourceDestCheck': True, 'StateReason': {'Code': 'Client.UserInitiatedShutdown', 'Message': 'Client.UserInitiatedShutdown: User initiated shutdown'}, 'Tags': [{'Key': 'Name', 'Value': 'ec2-test'}, {'Key': 'CreatorName', 'Value': '<REDACTED-EMAIL>'}], 'VirtualizationType': 'hvm', 'CpuOptions': {'CoreCount': 1, 'ThreadsPerCore': 1}, 'CapacityReservationSpecification': {'CapacityReservationPreference': 'open'}, 'HibernationOptions': {'Configured': False}, 'MetadataOptions': {'State': 'pending', 'HttpTokens': 'required', 'HttpPutResponseHopLimit': 1, 'HttpEndpoint': 'enabled', 'HttpProtocolIpv6': 'disabled', 'InstanceMetadataTags': 'disabled'}, 'EnclaveOptions': {'Enabled': False}, 'BootMode': 'uefi-preferred', 'PlatformDetails': 'Linux/UNIX', 'UsageOperation': 'RunInstances', 'UsageOperationUpdateTime': datetime.datetime(2024, 12, 19, 19, 58, 41, tzinfo=tzlocal()), 'PrivateDnsNameOptions': {'HostnameType': 'ip-name', 'EnableResourceNameDnsARecord': False, 'EnableResourceNameDnsAAAARecord': False}, 'MaintenanceOptions': {'AutoRecovery': 'default'}, 'CurrentInstanceBootMode': 'legacy-bios'}] 
[INFO]  2024-12-30T20:07:08.589Z  <REQUEST-ID>  Filtering resources using 0 filters 
[DEBUG]  2024-12-30T20:07:08.589Z  <REQUEST-ID>  Filtered from 1 to 1 ec2 
[INFO]  2024-12-30T20:07:08.589Z  <REQUEST-ID>  Filtered resources 1 of 1 
[DEBUG]  2024-12-30T20:07:08.605Z  <REQUEST-ID>  Storing output with <LogFile file:///tmp/ec2-deleted-or-created/custodian-run.log> 
[DEBUG]  2024-12-30T20:07:08.606Z  <REQUEST-ID>  metric:ResourceCount Count:1 policy:ec2-deleted-or-created restype:ec2 scope:policy 
[INFO]  2024-12-30T20:07:08.606Z  <REQUEST-ID>  Invoking actions [<c7n.actions.webhook.Webhook object at 0x7fb59a641bd0>] 
[INFO]  2024-12-30T20:07:08.607Z  <REQUEST-ID>  policy:ec2-deleted-or-created invoking action:webhook resources:1 
[INFO]  2024-12-30T20:07:09.081Z  <REQUEST-ID>  POST got response 400 with URL <REDACTED-URL>

You can pull information from the event field and the resource field for the C7N policy.
1. Accessing Fields
  Direct Access: Use the key name to access a field
  resource.InstanceId
2. Accessing Nested Fields
   Use . to navigate deeper into the structure
   resource.Placement.AvailabilityZone
3. Filtering Arrays
   Use [?condition] to filter arrays based on a condition.
   resource.Tags[?Key=='CreatorName']
   Access the Value directly:
   resource.Tags[?Key=='CreatorName'].Value


