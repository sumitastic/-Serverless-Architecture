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
