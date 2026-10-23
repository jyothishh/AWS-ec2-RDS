# AWS Infrastructure Project — Implementation Guide

## 1. Introduction

This document describes the implementation of an AWS infrastructure project using Amazon VPC, EC2, Application Load Balancer (ALB), EC2 Auto Scaling, Amazon RDS for MySQL, Amazon S3, AWS IAM, Amazon CloudFront, and Amazon CloudWatch.

The goal is to deploy a secure and scalable web infrastructure with load balancing, automatic scaling, managed database services, object storage, identity and access management, content delivery, monitoring, and backup capabilities.

The project demonstrates how AWS services can be integrated to provide secure storage, controlled access, application hosting, database connectivity, and infrastructure monitoring.

The document should be updated with actual resource names, configurations, screenshots, and test results from the AWS environment. A feature should be marked complete only after its implementation has been verified.

## 2. Prerequisites

The following are required:

* An AWS account with permission to create the required resources.
* Access to the AWS Management Console.
* Basic Linux command-line knowledge.
* Basic understanding of networking and security groups.
* An Ubuntu EC2 instance for the web server.
* An RDS MySQL database.
* An S3 bucket for application files and documents.
* An IAM role for EC2-to-S3 access.
* A GitHub repository for documentation and evidence.

Before beginning, review AWS pricing for the services you intend to use.

Avoid leaving unused EC2 instances, RDS databases, load balancers, NAT Gateways, or other billable resources running after testing.

## 3. Network Configuration

### Objective

Create a VPC environment in which web-facing resources and database resources can be organized and access can be controlled.

### Implementation Steps

1. Open the Amazon VPC console.
2. Create a VPC or identify the VPC already being used for the project.
3. Record the VPC ID and IPv4 CIDR block.
4. Identify the public and private subnets.
5. Review the subnet route tables.
6. Verify that the required internet access and routing are configured.
7. Review network ACLs if connectivity problems occur.

Web-facing components must have the network access required for their intended role. The database should remain private and should not be made publicly accessible merely to resolve connectivity problems.

The S3 bucket does not need to reside inside your VPC. Amazon S3 is a regional service. An EC2 instance can access it using its IAM role and the appropriate network connectivity.

For private networking, an S3 VPC endpoint can be considered as an additional enhancement. It is not required for the initial implementation.

### Security Group Considerations

Configure security groups according to the traffic flow:

* Load balancer to EC2: permit the required web traffic from the load balancer security group.
* EC2 administration: restrict SSH access to trusted IP addresses or use an appropriate managed access method.
* EC2 to RDS: permit MySQL traffic on TCP port 3306 from the EC2 security group.
* RDS inbound traffic: do not allow unrestricted access from the public internet.
* EC2 to S3: ensure that the instance has appropriate network connectivity to the S3 regional endpoint.

IAM permissions and security groups serve different purposes. Security groups control network traffic; IAM policies control access to AWS resources and API operations.

### Evidence to Collect

* VPC details.
* Public and private subnet details.
* Route tables.
* EC2 and RDS security group rules.
* S3 VPC endpoint configuration, if implemented.

Recommended folder: `screenshots/networking/`

## 4. EC2 and Apache Web Server

### Objective

Deploy a Linux web server that can receive requests from the Application Load Balancer.

### Implementation Steps

1. Open the Amazon EC2 console.
2. Launch an Ubuntu EC2 instance or use the existing project instance.
3. Select the correct VPC and subnet.
4. Attach the intended EC2 security group.
5. Connect to the instance using an authorized method.
6. Update the package index:

```bash
sudo apt update
```

7. Install Apache if it is not already installed:

```bash
sudo apt install apache2 -y
```

8. Start Apache and configure it to start automatically:

```bash
sudo systemctl enable --now apache2
```

9. Verify the service:

```bash
sudo systemctl status apache2
```

10. Check the local web server response:

```bash
curl -I http://localhost
```

11. If necessary, configure a simple test page for the project.
12. Verify that the EC2 security group permits web traffic from the load balancer security group.

Do not overwrite the default Apache page unless a custom page is required for the project.

### Evidence to Collect

* EC2 instance details.
* Apache service status showing it is active.
* Local HTTP response.
* Web page displayed through the intended access path.

Recommended folder: `screenshots/ec2-apache/`

## 5. Application Load Balancer

### Objective

Distribute incoming web traffic across registered EC2 targets.

### Implementation Steps

1. Open the EC2 console and select Load Balancers.
2. Create an Application Load Balancer.
3. Select the appropriate VPC.
4. Select the required subnets in suitable Availability Zones.
5. Attach the load balancer security group.
6. Create or select a target group for the EC2 web servers.
7. Configure the target type and health check path.
8. Register the EC2 instances or attach the Auto Scaling group.
9. Configure the required listener.
10. Wait for target health checks to complete.
11. Open the load balancer DNS name in a browser.
12. Troubleshoot unhealthy targets before proceeding.

A target must be healthy before it can normally receive traffic through the load balancer.

### Evidence to Collect

* Load balancer details.
* Listener configuration.
* Target group configuration.
* Target health showing healthy targets.
* Website response through the load balancer DNS name.

Recommended folder: `screenshots/load-balancer/`

## 6. EC2 Auto Scaling

### Objective

Configure EC2 Auto Scaling to maintain the desired instance capacity and adjust capacity based on demand.

### Implementation Steps

1. Create a launch template containing the required EC2 configuration.
2. Configure the AMI, instance type, security group, and other required settings.
3. Ensure the launch template installs or starts Apache on newly launched instances.
4. Create an Auto Scaling group using the launch template.
5. Select the appropriate VPC subnets.
6. Configure the minimum, desired, and maximum instance counts.
7. Attach the Auto Scaling group to the load balancer target group.
8. Configure a target tracking scaling policy.
9. Set the target average CPU utilization to 50% if that is the chosen project requirement.
10. Review the associated CloudWatch alarms and scaling activities.
11. Verify that newly launched instances register with the target group and pass health checks.

A target of 50% CPU utilization is a scaling target, not a guarantee that a new instance launches immediately whenever CPU exceeds 50%. Scaling behavior depends on metrics, alarm evaluation, instance warm-up, and capacity limits.

If the EC2 application requires S3 access, ensure that instances launched through the Auto Scaling group receive the correct IAM instance profile. Configure this in the launch template so that newly launched instances inherit the required role.

### Evidence to Collect

* Launch template details.
* Auto Scaling group configuration.
* Minimum, desired, and maximum capacity.
* Scaling policy.
* CloudWatch alarm details.
* Scaling activity history.
* Newly launched instance and its target health, if scaling is successfully tested.
* IAM instance profile configuration in the launch template.

Recommended folder: `screenshots/auto-scaling/`

## 7. Amazon RDS for MySQL

### Objective

Deploy a managed MySQL database and allow authorized application traffic from the EC2 environment.

### Implementation Steps

1. Open the Amazon RDS console.

2. Create an RDS MySQL database or identify the existing project database.

3. Record the database identifier and endpoint.

4. Confirm that the database is in the intended VPC.

5. Review the DB subnet group and network placement.

6. Confirm that the database status is `Available`.

7. Identify the security group attached to the RDS instance.

8. Add an inbound security group rule for MySQL:

   * Type: MySQL/Aurora
   * Protocol: TCP
   * Port: 3306
   * Source: the EC2 instance's security group

9. Keep public accessibility disabled unless a separate, justified requirement dictates otherwise.

10. Store the database username and password securely.

The EC2 and RDS instances being in the same VPC does not automatically guarantee connectivity. Security groups, routing, network ACLs, and database status must also permit the connection.

### Evidence to Collect

* RDS database details.
* Database status.
* Database endpoint and port, with sensitive details handled appropriately.
* DB subnet group and VPC details.
* RDS security group inbound rule.

Recommended folder: `screenshots/rds/`

## 8. Connect EC2 to RDS

### Objective

Verify network connectivity and establish a MySQL session from the EC2 instance.

### Implementation Steps

#### Step 1: Verify DNS Resolution

On the EC2 instance, run:

```bash
getent hosts YOUR_RDS_ENDPOINT
```

Replace `YOUR_RDS_ENDPOINT` with the actual RDS endpoint.

#### Step 2: Test TCP Connectivity

Run:

```bash
nc -vz -w 5 YOUR_RDS_ENDPOINT 3306
```

If the connection times out, investigate the RDS security group first. Confirm that its inbound rule allows TCP port 3306 from the correct EC2 security group.

Also check EC2 outbound rules and, if necessary, subnet routing and network ACLs.

Do not treat a timeout as a password problem. Authentication should be investigated after network connectivity succeeds.

#### Step 3: Install a MySQL Client if Needed

On Ubuntu:

```bash
sudo apt update
sudo apt install mysql-client -y
```

If the package is unavailable under that name in the selected Ubuntu release, check the available MySQL client package for that release.

#### Step 4: Connect to MySQL

Use the actual RDS endpoint and an authorized database account:

```bash
mysql -h YOUR_RDS_ENDPOINT -P 3306 -u admin -p
```

Enter the database password when prompted. Do not put the password directly in the command or commit it to GitHub.

If the project uses TLS certificate verification, use the appropriate RDS CA bundle and SSL settings for the installed MySQL client.

#### Step 5: Verify the Database Session

After a successful connection, run:

```sql
SELECT VERSION();

SHOW DATABASES;
```

If you have permission to create a test database, you can also run:

```sql
CREATE DATABASE employee_db;

SHOW DATABASES;
```

Only create resources that are appropriate for your database and permissions. Clean up test databases if they are no longer needed.


## 9. Amazon S3 — Object Storage

### Objective

Create a secure S3 bucket to store project files, images, screenshots, documents, or application assets.

For the current project, the bucket can contain the resume PDF and two screenshots already selected for upload. These files demonstrate object storage and can be used to test the EC2 instance's access permissions.

If the project later includes a functioning employee application, the bucket can also store sample employee images or other static assets.

### Implementation Steps

#### Step 1: Create an S3 Bucket

1. Open the Amazon S3 console.
2. Click **Create bucket**.
3. Enter a globally unique bucket name.
4. Select an appropriate AWS Region, preferably the same Region as the EC2 instance.
5. Keep Block Public Access enabled.
6. Enable bucket versioning.
7. Leave other settings at their defaults unless the project requires changes.
8. Create the bucket.

Record the bucket name and Region in the project documentation.

#### Step 2: Upload Project Files

1. Open the newly created bucket.
2. Click **Upload**.
3. Select the resume PDF and two screenshots.
4. Upload the files.
5. Verify that all three objects appear in the bucket.

The object names can be chosen freely. Meaningful filenames make the project easier to document.

Do not upload confidential information, passwords, AWS access keys, or other sensitive files. Keep the bucket private because the resume contains personal information.

#### Step 3: Configure Bucket Security

Open the bucket's Permissions tab.

Verify that:

* Block Public Access is enabled.
* No public bucket policy has been configured.
* Access is granted only to the intended AWS identities.
* Versioning is enabled.

Avoid adding public-read ACLs or policies.

#### Step 4: Understand S3 Object Permissions

An S3 bucket contains objects identified by their object keys. IAM permissions determine which operations an identity can perform.

Examples include:

* `s3:ListBucket` — list objects in the bucket.
* `s3:GetObject` — retrieve objects.
* `s3:PutObject` — upload objects.

These permissions should be granted only when needed.


## 10. AWS IAM — Secure EC2-to-S3 Access

### Objective

Create an IAM role that allows the EC2 instance to access only the intended S3 bucket.

This demonstrates the principle of least privilege and the use of temporary credentials provided through an EC2 instance profile.

### Implementation Steps

#### Step 1: Create a Custom IAM Policy

1. Open IAM.
2. Select **Policies → Create policy → JSON**.
3. Enter the following policy, replacing `YOUR_BUCKET_NAME` with the actual S3 bucket name.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListProjectBucket",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME"
    },
    {
      "Sid": "ReadAndUploadProjectObjects",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}
```

4. Name the policy `EC2S3ProjectAccessPolicy`.
5. Create the policy.

This policy permits listing the bucket, reading objects, and uploading objects. It does not grant permission to delete objects.

The policy is intentionally scoped to one bucket. If the project only requires read access, remove `s3:PutObject`. If it only requires uploads, remove `s3:GetObject`.

#### Step 2: Create the IAM Role

1. In IAM, select **Roles → Create role**.
2. Select **AWS service** as the trusted entity type.
3. Choose **EC2** as the service or use case.
4. Attach `EC2S3ProjectAccessPolicy`.
5. Name the role `EC2S3ProjectRole`.
6. Review the configuration and create the role.

The EC2 trust relationship allows the EC2 service to assume the role.

#### Step 3: Attach the Role to EC2

1. Open the EC2 console.
2. Select the application instance.
3. Choose **Actions → Security → Modify IAM role**.
4. Select `EC2S3ProjectRole`.
5. Update the IAM role.

If the application uses an Auto Scaling group, also configure the IAM instance profile in the launch template. Otherwise, newly launched instances may not receive the intended permissions.

#### Step 4: Verify IAM Role Attachment

Return to the EC2 instance details and confirm that the correct IAM role is attached.

Do not create permanent AWS access keys for the instance. The instance profile provides credentials automatically when the role is configured correctly.

### Evidence to Collect

* Custom IAM policy JSON.
* IAM role details.
* Role trust relationship showing EC2.
* EC2 instance details showing the attached role.
* Launch template instance profile configuration, if applicable.

Recommended folder: `screenshots/iam/`

## 11. Amazon CloudFront

### Objective

Configure CloudFront to deliver web content through a content delivery network, using the Application Load Balancer as the existing application origin.

### Implementation Steps

1. Open the Amazon CloudFront console.
2. Create or inspect the existing distribution.
3. Configure the Application Load Balancer as the application origin.
4. Select the correct origin protocol policy.
5. Configure the viewer protocol policy, such as redirecting HTTP requests to HTTPS.
6. Review the cache policy.
7. Use suitable caching behavior for the website's content.
8. Avoid aggressive caching for dynamic content unless it has been designed and tested for caching.
9. Create or update the distribution.
10. Wait until the distribution status indicates it is deployed.
11. Test the website using the CloudFront distribution domain.
12. Verify that the response is correct and the origin is reachable.

For an HTTPS origin, the load balancer must have a suitable certificate and HTTPS listener.

CloudFront does not replace the load balancer or repair unhealthy EC2 targets. Confirm that the origin works before troubleshooting CloudFront.


## 12. Amazon CloudWatch Monitoring

### Objective

Monitor infrastructure performance and observe events that affect scaling and availability.

### Implementation Steps

1. Open the CloudWatch console.
2. Review EC2 CPU utilization.
3. Review load balancer metrics.
4. Review target group health-related metrics.
5. Review RDS metrics such as CPU utilization and database connections.
6. Inspect the alarms associated with Auto Scaling.
7. Configure additional alarms if required.
8. Record the alarm thresholds and evaluation settings.
9. If testing scaling, observe the metric and resulting Auto Scaling activity.


## 13. RDS Backups and Snapshots

### Objective

Understand how database backups support recovery.

### Implementation Steps

1. Open the RDS console.
2. Select the database.
3. Review its automated backup retention settings.
4. Record the backup window and retention period.
5. Review available automated backups.
6. If required, create a manual DB snapshot.
7. Record the snapshot identifier and status.
8. Understand the process for restoring a snapshot to a new database instance.


## 14. Final Validation

Before considering the project complete, verify the following:

### Networking and Web Hosting

* [ ] VPC and subnet configuration is documented.
* [ ] Apache is active on the EC2 instance.
* [ ] The load balancer has healthy targets.
* [ ] The website is accessible through the load balancer.

### Auto Scaling

* [ ] Auto Scaling settings are documented.
* [ ] The scaling policy and associated alarms are verified.
* [ ] Newly launched instances receive the intended IAM instance profile, if applicable.

### Database

* [ ] RDS is available and appropriately private.
* [ ] EC2-to-RDS TCP connectivity succeeds.
* [ ] MySQL login and SQL queries succeed.

### S3 and IAM

* [ ] The S3 bucket has been created.
* [ ] Project files have been uploaded.
* [ ] Block Public Access is enabled.
* [ ] Bucket versioning is enabled.
* [ ] The custom IAM policy is scoped to the intended bucket.
* [ ] The EC2 IAM role has the correct trust relationship.
* [ ] The role is attached to the EC2 instance.
* [ ] The AWS CLI identifies the assumed role.
* [ ] Bucket listing succeeds.
* [ ] Object download succeeds when permitted.
* [ ] An unauthorized operation is denied.
* [ ] No permanent AWS access keys are stored on the EC2 instance.

### CloudFront

* [ ] CloudFront distribution is deployed.
* [ ] The website is accessible through CloudFront.
* [ ] The S3 origin and OAC are verified if the optional enhancement is implemented.

### Monitoring and Backups

* [ ] CloudWatch metrics and alarms are documented.
* [ ] RDS backup settings are verified.
* [ ] Snapshot or restore evidence is included if implemented.

### Documentation

* [ ] Required screenshots are organized in GitHub.
* [ ] Implementation status reflects the actual test results.
* [ ] No passwords, access keys, private keys, or confidential logs are committed.


## 15. Conclusion

This project demonstrates practical experience with AWS networking, Linux web server administration, load balancing, Auto Scaling, managed databases, object storage, identity and access management, content delivery, monitoring, backups, and troubleshooting.

The S3 and IAM implementation demonstrates secure object storage and role-based access for an EC2 workload without requiring permanent access keys on the instance.

The optional CloudFront S3 origin demonstrates how static content can be delivered while keeping the storage bucket private.

The final project report should distinguish between resources that were configured, features that were tested successfully, and features that remain incomplete.
