# Configure and Test a VPC Endpoint

This project showcases the setup of a VPC endpoint in AWS, validation of its connectivity to an S3 bucket via an EC2 instance, and implementation of a bucket policy to enforce access restrictions exclusively through the VPC endpoint.

## Prerequisites

- An existing VPC named `test-vpc` with both public and private subnets.
- An S3 bucket named `my-test-s3-bucket`.
- AWS CLI configured on your local machine.

![s3 endpoint design](https://github.com/user-attachments/assets/b7d5ca7f-96c7-4a52-9190-e29b08b215d5)

## Steps

### Step 1: Create a VPC Endpoint

1. **Navigate to the VPC Dashboard:**
   - Open the AWS Management Console and navigate to the VPC Dashboard.

2. **Create VPC Endpoint:**
   - Select "Endpoints" from the left-hand menu.
   - Click "Create Endpoint".
   - In the "Name tag" field (optional), enter the name for your endpoint. For example, you could use "my-test-endpoint". 
   - Choose the AWS service `com.amazonaws.<region>.s3` (replace `<region>` with your specific region, e.g., `com.amazonaws.us-east-1.s3`).
   - Select "Gateway" as the type.
   - Choose `test-vpc` under the VPC section.
   - Select the route table associated with the VPC.
   - Grant full access permission.

3. **Edit Route Table:**
   - Navigate to the Route Tables section within the VPC Dashboard.
   - Select the route table associated with `test-vpc`.
   - Add a route with the destination as `pl-63a5400a` (prefix list ID for the S3 service in the selected region) and the target as the VPC endpoint.

### Step 2: Configure S3 Bucket Policy

1. **Open S3 Bucket Policy Generator:**
   - Navigate to the S3 service in the AWS Management Console.
   - Select the bucket `my-test-s3-bucket`.
   - Go to the "Permissions" tab and click "Bucket Policy".

2. **Create Bucket Policy:**
   - Use the S3 Bucket Policy Generator to create a policy with the following details:
     - **Effect:** Deny
     - **AWS Service:** Amazon S3
     - **Actions:** All Actions
     - **Amazon Resource Name (ARN):** arn:aws:s3:::my-test-s3-bucket/*
     - **Add Conditions:**
       - **Condition:** StringNotEquals
       - **Key:** aws:SourceVpce
       - **Value:** VPC endpoint ID (e.g., `vpce-12345`)

3. **Attach Policy:**
   - Add the generated policy to the bucket policy and save.

### Step 3: Create and Configure EC2 Instance

1. **Launch EC2 Instance:**
   - Navigate to the EC2 Dashboard in the AWS Management Console.
   - Click "Launch Instance".
   - Choose an Amazon Machine Image (AMI).
   - Select an instance type (e.g., t2.micro).
   - Configure the instance to be within the public subnet of `test-vpc`.

2. **Create IAM Role:**
   - Navigate to the IAM Dashboard.
   - Create a role named `test-ec2-s3-role` with the policy `AmazonS3FullAccess`.
   - Attach this role to the EC2 instance during the launch process.

### Step 4: Test VPC Endpoint from EC2 Instance

1. **Connect to EC2 Instance:**
   - Use SSH to connect to the EC2 instance.

2. **Test S3 Access:**
   - Run the following command to list the S3 buckets:
     ```sh
     aws s3 ls
     ```
   - To list the contents of the specific bucket, run:
     ```sh
     aws s3 ls s3://my-test-s3-bucket
     ```

### Step 5: Remove Bucket Policy (if needed)

- If you need to remove the bucket policy, run the following command from the AWS CLI:
  ```sh
  aws s3api delete-bucket-policy --bucket my-test-s3-bucket
