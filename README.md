import boto3

def lambda_handler(event, context):
    # Initialize a boto3 EC2 client
    ec2_client = boto3.client('ec2')
    
    # Describe instances with 'Auto-Stop' and 'Auto-Start' tags
    auto_stop_instances = ec2_client.describe_instances(Filters=[{'Name': 'tag:Action', 'Values': ['Auto-Stop']}])
    auto_start_instances = ec2_client.describe_instances(Filters=[{'Name': 'tag:Action', 'Values': ['Auto-Start']}])

    # Stop the 'Auto-Stop' instances
    for reservation in auto_stop_instances['Reservations']:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            ec2_client.stop_instances(InstanceIds=[instance_id])
            print(f"Stopped instance: {instance_id}")

    # Start the 'Auto-Start' instances
    for reservation in auto_start_instances['Reservations']:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            ec2_client.start_instances(InstanceIds=[instance_id])
            print(f"Started instance: {instance_id}")

    return {
        'statusCode': 200,
        'body': 'EC2 instances auto-started and auto-stopped successfully.'
    }
    
    #Question 2 solution##
    
    import boto3
from datetime import datetime, timedelta

def lambda_handler(event, context):
    # Initialize a boto3 S3 client
    s3_client = boto3.client('s3')
    
    # Specify the S3 bucket name
    bucket_name = 'my-automated-bucket'  # Replace with your bucket name
    
    # List objects in the specified bucket
    objects = s3_client.list_objects(Bucket=bucket_name)

    # Calculate the date threshold (30 days ago)
    threshold_date = datetime.utcnow() - timedelta(days=30)

    # Delete objects older than 30 days
    deleted_objects = []
    for obj in objects.get('Contents', []):
        last_modified = obj['LastModified']
        if last_modified < threshold_date:
            s3_client.delete_object(Bucket=bucket_name, Key=obj['Key'])
            deleted_objects.append(obj['Key'])
    
    # Print the names of deleted objects for logging purposes
    if deleted_objects:
        print(f"Deleted objects: {deleted_objects}")
    else:
        print("No objects deleted.")

    return {
        'statusCode': 200,
        'body': 'S3 bucket cleanup completed successfully.'
    }
#Question 3:

import boto3

def lambda_handler(event, context):
    # Initialize a boto3 RDS client
    rds_client = boto3.client('rds')
    
    # Specify the RDS instance identifier
    rds_instance_identifier = 'your-rds-instance-id'  # Replace with your RDS instance ID
    
    # Take a snapshot of the specified RDS instance
    snapshot_response = rds_client.create_db_snapshot(
        DBSnapshotIdentifier=f'RDS-Snapshot-{rds_instance_identifier}-{event["time"]}',
        DBInstanceIdentifier=rds_instance_identifier
    )

    # Print the snapshot ID for logging purposes
    print(f"Snapshot ID: {snapshot_response['DBSnapshot']['DBSnapshotIdentifier']}")

    return {
        'statusCode': 200,
        'body': 'RDS snapshot created successfully.'
    }
#Question-4

import boto3

def lambda_handler(event, context):
    # Initialize a boto3 S3 client
    s3_client = boto3.client('s3')
    
    # List all S3 buckets
    buckets = s3_client.list_buckets()['Buckets']

    # Detect buckets without server-side encryption
    unencrypted_buckets = []
    for bucket in buckets:
        bucket_name = bucket['Name']
        bucket_encryption = s3_client.get_bucket_encryption(Bucket=bucket_name)
        
        # Check if server-side encryption is not enabled
        if 'ServerSideEncryptionConfiguration' not in bucket_encryption:
            unencrypted_buckets.append(bucket_name)
    
    # Print the names of unencrypted buckets for logging purposes
    if unencrypted_buckets:
        print(f"Unencrypted buckets: {unencrypted_buckets}")
    else:
        print("All buckets are encrypted.")

    return {
        'statusCode': 200,
        'body': 'S3 bucket encryption check completed.'
    }

