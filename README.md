

# Cloud Cost Optimization with AWS Lambda

## Overview
This project implements a cost optimization strategy for AWS by identifying and deleting unused Elastic Block Store (EBS) snapshots. By automating the cleanup process with AWS Lambda, we can ensure that unnecessary storage costs are minimized.

## Architecture
The solution leverages the following AWS services:
- **AWS Lambda**: Executes the snapshot cleanup code.
- **Amazon EC2**: Provides the snapshots to be cleaned up.
- **CloudWatch Events**: Schedules the Lambda function to run periodically.

## Prerequisites
- AWS Account with appropriate permissions
- AWS CLI configured on your local machine
- Basic knowledge of AWS services (EC2, Lambda, CloudWatch)

## Setup Instructions

### 1. IAM Role for Lambda

1. **Create IAM Role**:
   - Navigate to the [IAM Console](https://console.aws.amazon.com/iam/).
   - Create a new role with the following policies:
     - `AmazonEC2ReadOnlyAccess`
     - `AmazonEBSFullAccess`
     - `CloudWatchLogsFullAccess`
   - Name the role `Lambda_EBS_Cleanup_Role`.

### 2. Create the Lambda Function

1. **Navigate to the Lambda Console**:
   - Open the [AWS Lambda Console](https://console.aws.amazon.com/lambda/).
   - Create a function from scratch with the following settings:
     - Name: `EBS_Snapshot_Cleanup`
     - Runtime: `Python 3.x`
     - Role: `Lambda_EBS_Cleanup_Role`

2. **Add the Lambda Function Code**:
   - Copy and paste the following code into the Lambda function editor:


```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')

    # Get all EBS snapshots
    response = ec2.describe_snapshots(OwnerIds=['self'])

    # Get all active EC2 instance IDs
    instances_response = ec2.describe_instances(Filters=[{'Name': 'instance-state-name', 'Values': ['running']}])
    active_instance_ids = set()

    for reservation in instances_response['Reservations']:
        for instance in reservation['Instances']:
            active_instance_ids.add(instance['InstanceId'])

    # Iterate through each snapshot and delete if it's not attached to any volume or the volume is not attached to a running instance
    for snapshot in response['Snapshots']:
        snapshot_id = snapshot['SnapshotId']
        volume_id = snapshot.get('VolumeId')

        if not volume_id:
            # Delete the snapshot if it's not attached to any volume
            ec2.delete_snapshot(SnapshotId=snapshot_id)
            print(f"Deleted EBS snapshot {snapshot_id} as it was not attached to any volume.")
        else:
            # Check if the volume still exists
            try:
                volume_response = ec2.describe_volumes(VolumeIds=[volume_id])
                if not volume_response['Volumes'][0]['Attachments']:
                    ec2.delete_snapshot(SnapshotId=snapshot_id)
                    print(f"Deleted EBS snapshot {snapshot_id} as it was taken from a volume not attached to any running instance.")
            except ec2.exceptions.ClientError as e:
                if e.response['Error']['Code'] == 'InvalidVolume.NotFound':
                    # The volume associated with the snapshot is not found (it might have been deleted)
                    ec2.delete_snapshot(SnapshotId=snapshot_id)
                    print(f"Deleted EBS snapshot {snapshot_id} as its associated volume was not found.")
                                                

```

3. **Deploy the Lambda Function**:
   - Click `Deploy` to save and deploy the changes.

### 3. Test the Lambda Function

1. **Create a Test Event**:
   - Click `Test` and configure a new test event with default values.
   - Execute the test and check the logs for output.

### 4. Schedule the Lambda Function

1. **Create a CloudWatch Events Rule**:
   - Open the [CloudWatch Console](https://console.aws.amazon.com/cloudwatch/).
   - Create a new rule with the following settings:
     - Event Source: `Schedule`
     - Schedule: `cron(0 0 * * ? *)` (to run daily)

2. **Add Target**:
   - Add the Lambda function `EBS_Snapshot_Cleanup` as the target.

3. **Complete the Rule Creation**:
   - Name the rule and complete the setup.

## Monitoring and Logging

- **CloudWatch Logs**: Ensure that the Lambda function is logging to CloudWatch Logs for monitoring and debugging.
- **CloudWatch Metrics**: Monitor Lambda invocation metrics to ensure that the function is running as expected.

## Conclusion
This setup ensures that unused EBS snapshots are regularly cleaned up, reducing storage costs. You can further customize the Lambda function and schedule based on your specific requirements.

## Author
- **Aakash Kumar**  
  - [LinkedIn](https://www.linkedin.com/in/aakash-kumar-251ab9185/)
  - [GitHub](https://www.github.com/Aakash644)

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
```

### Instructions

1. **Create a new GitHub repository**: 
   - Go to GitHub and create a new repository named `cloud-cost-optimization`.

2. **Add the README file**:
   - Copy the above README content and create a `README.md` file in your repository.
   - Commit and push the `README.md` file to your repository.

3. **Upload your Lambda function code**:
   - Create a directory named `lambda_function` and add your `lambda_function.py` file with the provided code.

4. **Commit and Push**:
   - Commit your changes and push them to your GitHub repository.
