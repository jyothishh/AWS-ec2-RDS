# AWS-EC2-RDS

## Project Overview

This project demonstrates the design and implementation of an AWS web infrastructure using core AWS networking, compute, database, storage, load balancing, content delivery, monitoring, and backup services.

The infrastructure is designed to distribute incoming web traffic across EC2 instances, scale compute capacity based on demand, store database information in Amazon RDS, securely store files in Amazon S3, and deliver website content through Amazon CloudFront.

AWS IAM roles and policies provide controlled access to AWS resources, following the principle of least privilege.

The primary focus of this project is **AWS infrastructure implementation, configuration, security, monitoring, availability, and troubleshooting**, rather than application development.

## Project Objectives

* Build a custom Amazon VPC environment.
* Configure public and private subnets.
* Deploy Ubuntu EC2 instances running Apache.
* Configure an Application Load Balancer to distribute traffic.
* Implement EC2 Auto Scaling based on CPU utilization.
* Deploy an Amazon RDS MySQL database.
* Configure secure connectivity between EC2 and RDS.
* Create an Amazon S3 bucket for secure object storage.
* Configure an IAM role for secure EC2-to-S3 access.
* Implement least-privilege IAM permissions.
* Enable S3 Block Public Access and bucket versioning.
* Configure Amazon CloudFront with the load balancer as its origin.
* Optionally integrate S3 as a CloudFront origin for static content.
* Monitor infrastructure using Amazon CloudWatch.
* Understand RDS automated backups and manual snapshots.
* Document implementation steps, testing results, and troubleshooting.

## AWS Services Used

| AWS Service                  | Purpose                                                  |
| ---------------------------- | -------------------------------------------------------- |
| Amazon VPC                   | Provides an isolated virtual network                     |
| Public and private subnets   | Organize resources according to network accessibility    |
| Security Groups              | Control inbound and outbound network traffic             |
| Amazon EC2                   | Hosts the Apache web server                              |
| Application Load Balancer    | Distributes incoming HTTP/HTTPS requests                 |
| EC2 Auto Scaling             | Adjusts the number of EC2 instances according to demand  |
| Amazon RDS for MySQL         | Provides a managed relational database                   |
| Amazon S3                    | Stores files, documents, images, and other objects       |
| AWS IAM                      | Controls access to AWS resources                         |
| IAM Roles                    | Provide temporary AWS credentials to authorized services |
| IAM Policies                 | Define permitted and denied AWS API operations           |
| Amazon CloudFront            | Delivers content through a content delivery network      |
| Amazon CloudWatch            | Monitors metrics, alarms, and infrastructure activity    |
| Amazon RDS automated backups | Supports database recovery                               |
| Amazon RDS snapshots         | Provides database copies for recovery scenarios          |

## Architecture

The intended request flow is:

```text
                         Users
                           |
                           v
                    Amazon CloudFront
                           |
                           v
               Application Load Balancer
                           |
                 +---------+---------+
                 |                   |
                 v                   v
            EC2 Instance 1      EC2 Instance 2
            Apache Web Server   Apache Web Server
                 |                   |
                 +---------+---------+
                           |
                           v
                    Amazon RDS MySQL
                    Private Database


                 EC2 Instances
                       |
                       | IAM Instance Profile
                       | and IAM Role
                       v
                 Amazon S3 Bucket
                 Private Object Storage
                       |
             +---------+----------+
             |                    |
             v                    v
        Project Files        Images/Documents
        Resume PDF           Other Objects


        Amazon CloudWatch
        Monitors Infrastructure

        EC2 Auto Scaling
        Adjusts Compute Capacity

        RDS Automated Backups
        and Manual Snapshots
```

The EC2 instances access the private S3 bucket using an IAM role attached through an instance profile. The role grants only the required S3 permissions.

The S3 bucket is separate from the VPC and does not require a public IP address.

The primary CloudFront distribution uses the Application Load Balancer as its application origin. S3 can optionally be configured as a second CloudFront origin to deliver static objects.

**Note:** This diagram represents the intended architecture. Confirm actual subnet placement, connectivity, target health, IAM permissions, and service status before presenting the project as fully operational.

## Implementation Phases

### Phase 1: Networking

* Create or configure the VPC.
* Configure public and private subnets.
* Configure route tables and internet access where required.
* Configure security groups with restricted access.
* Verify the network connectivity required by the EC2, RDS, and load-balancing components.

### Phase 2: EC2 and Apache

* Launch an Ubuntu EC2 instance.
* Install and start Apache.
* Deploy a simple web page.
* Verify that Apache is running.
* Record the instance details and web server configuration.

### Phase 3: Application Load Balancer

* Create an Application Load Balancer.
* Configure a target group.
* Register the EC2 instances.
* Configure the listener.
* Verify that registered targets are healthy.
* Test the website through the load balancer DNS name.

### Phase 4: Auto Scaling

* Create a launch template.
* Configure an Auto Scaling group.
* Attach the group to the target group.
* Configure minimum, desired, and maximum instance capacity.
* Configure a target tracking scaling policy with a 50% average CPU utilization target.
* Verify scaling activity when suitable load and metric conditions are met.
* Configure the required IAM instance profile in the launch template if the instances need S3 access.

### Phase 5: Amazon RDS

* Deploy an RDS MySQL database.
* Configure the database in the intended VPC and subnet arrangement.
* Restrict database access through security groups.
* Configure the EC2 security group as the source of permitted MySQL traffic.
* Test connectivity from EC2 to RDS.
* Run basic SQL commands after successfully connecting.

### Phase 6: Amazon S3 Object Storage

**Objective:** Store project files securely in an S3 bucket.

Implementation steps:

1. Open the Amazon S3 console.
2. Create a bucket with a globally unique name.
3. Select an appropriate AWS Region.
4. Keep Block Public Access enabled.
5. Enable bucket versioning.
6. Upload the project resume PDF and two screenshots, or other suitable test files.
7. Verify that the uploaded objects appear in the bucket.
8. Review the bucket's Properties and Permissions settings.
9. Record the bucket name, Region, and object keys in the project documentation.

The bucket should remain private. The resume and screenshots are sample objects for demonstrating S3 storage and should not be exposed through public access.

### Phase 7: IAM Role and EC2-to-S3 Access

**Objective:** Configure secure access to the S3 bucket without storing permanent AWS access keys on the EC2 instance.

#### Step 1: Create an IAM Policy

Create a custom IAM policy named `EC2S3ProjectAccessPolicy`.

Replace `YOUR_BUCKET_NAME` with the actual bucket name.

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
      "Sid": "AccessProjectObjects",
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

This policy allows the role to list objects, download objects, and upload objects in the designated bucket.

It does not grant permission to delete objects.

If the project only requires read access, remove `s3:PutObject`. Grant only the permissions needed for the intended operations.

#### Step 2: Create the IAM Role

1. Open IAM → Roles.
2. Click Create role.
3. Select AWS service as the trusted entity type.
4. Choose EC2 as the service or use case.
5. Attach `EC2S3ProjectAccessPolicy`.
6. Name the role `EC2S3ProjectRole`.
7. Review and create the role.

The role's trust relationship allows EC2 to assume the role.

#### Step 3: Attach the Role to EC2

1. Open the EC2 console.
2. Select the application instance.
3. Choose Actions → Security → Modify IAM role.
4. Select `EC2S3ProjectRole`.
5. Update the IAM role.

If the application uses an Auto Scaling group, configure the same instance profile in its launch template so newly launched instances receive the role.

#### Step 4: Verify EC2-to-S3 Access

Connect to the EC2 instance and check whether the AWS CLI is available:

```bash
aws --version
```

Verify the identity used by the instance:

```bash
aws sts get-caller-identity
```

The returned identity should represent the assumed EC2 role.

List the objects in the project bucket:

```bash
aws s3 ls s3://YOUR_BUCKET_NAME
```

Download a test object:

```bash
aws s3 cp s3://YOUR_BUCKET_NAME/resume.pdf /tmp/resume-test.pdf
```

Replace the object name if your uploaded file has a different key.

Expected results:

* The caller identity shows the assumed role.
* The bucket listing succeeds.
* The permitted object download succeeds.

To demonstrate least privilege, attempt an unauthorized operation against an expendable test object:

```bash
aws s3 rm s3://YOUR_BUCKET_NAME/test-object.txt
```

The operation should be denied if no other applicable policy grants delete permission. Do not test against important files.

If the AWS CLI is not installed, install the appropriate version for Ubuntu using the official AWS installation instructions.

### Phase 8: CloudFront

* Create or inspect a CloudFront distribution.
* Configure the Application Load Balancer as the application origin.
* Configure the required cache behavior.
* Wait until the distribution is deployed.
* Test the website through the CloudFront distribution domain.
* Verify the behavior of dynamic content before enabling aggressive caching.

#### Optional: Add S3 as a CloudFront Origin

S3 can be configured as an additional origin for static content.

Implementation steps:

1. Add the S3 bucket as an origin.
2. Configure Origin Access Control (OAC).
3. Keep S3 Block Public Access enabled.
4. Configure the bucket policy to permit the appropriate CloudFront distribution to retrieve objects.
5. Create a cache behavior for the relevant path pattern.
6. Deploy the distribution.
7. Test the object through CloudFront.

This feature is optional. The initial project can use S3 through the EC2 IAM role without changing the existing CloudFront origin.

### Phase 9: Monitoring and Backups

* Review EC2 CPU utilization in CloudWatch.
* Configure or inspect scaling alarms.
* Review load balancer and target group metrics.
* Review RDS monitoring metrics.
* Verify RDS automated backup configuration.
* Create a manual database snapshot if required.
* Document backup and recovery considerations.

## Testing and Verification

Update this table as you complete and verify each test.

| Test                    | Expected Result                                       | Status               |
| ----------------------- | ----------------------------------------------------- | -------------------- |
| Apache service          | Apache is active and serving the intended page        | Pending verification |
| Load balancer           | Requests reach healthy registered targets             | Pending verification |
| Auto Scaling            | Group configuration and scaling policy are correct    | Pending verification |
| EC2-to-RDS connectivity | EC2 connects to MySQL on TCP port 3306                | In progress          |
| SQL operations          | Basic SQL queries execute successfully                | Pending verification |
| S3 bucket               | Bucket exists with Block Public Access enabled        | Pending verification |
| S3 versioning           | Bucket versioning is enabled                          | Pending verification |
| IAM role                | EC2S3ProjectRole is attached to the instance          | Pending verification |
| IAM permissions         | Policy grants only the intended S3 operations         | Pending verification |
| EC2-to-S3 access        | Bucket listing and object download succeed            | Pending verification |
| S3 access control       | Unauthorized object deletion is denied                | Pending verification |
| CloudFront              | Website is accessible through the distribution domain | Pending verification |
| CloudWatch              | Relevant metrics and alarms are visible               | Pending verification |
| RDS backups             | Backup configuration is verified                      | Pending verification |

Only mark a test as completed after collecting evidence that demonstrates the expected result.

## Security Considerations

* Avoid exposing the database directly to the public internet.
* Allow MySQL traffic on TCP port 3306 only from the required EC2 security group.
* Restrict SSH access to trusted IP addresses.
* Keep S3 Block Public Access enabled.
* Use private S3 buckets for confidential or personal documents.
* Follow the principle of least privilege when assigning IAM permissions.
* Use IAM roles and temporary credentials for EC2-to-S3 access.
* Avoid storing permanent AWS access keys on EC2 instances.
* Avoid storing database passwords or private keys in GitHub.
* Use HTTPS where configured and appropriate.
* Review security group rules and network access controls.
* Review applicable IAM and bucket policies when troubleshooting access errors.


## Skills Demonstrated

* AWS infrastructure deployment.
* VPC and subnet configuration.
* Security group configuration.
* EC2 and Apache administration.
* Application Load Balancer configuration.
* EC2 Auto Scaling.
* Amazon RDS and MySQL connectivity.
* Amazon S3 object storage.
* IAM roles and policies.
* Least-privilege access control.
* Secure EC2-to-S3 integration.
* CloudFront configuration.
* CloudWatch monitoring.
* Database backup concepts.
* Linux command-line administration.
* Infrastructure troubleshooting.
* Technical documentation using GitHub.

## Cost Management

AWS resources may incur charges even when used for learning.

* Stop or terminate resources when no longer required.
* Review EC2, load balancer, RDS, CloudFront, storage, and data transfer charges.
* Review applicable charges for snapshots, logging, and optional network endpoints.
* Remove unused snapshots and other resources when no longer needed.
* Review AWS pricing and Free Tier or credit eligibility before deploying resources.

## Conclusion

This project demonstrates practical knowledge of designing, deploying, monitoring, and troubleshooting AWS infrastructure using managed cloud services.

It emphasizes infrastructure administration, availability, scaling, database connectivity, secure object storage, IAM access control, content delivery, monitoring, and operational documentation.

**Project status:** Update this section as implementation and testing phases are completed. Distinguish between configured resources, successfully tested features, and incomplete components.
