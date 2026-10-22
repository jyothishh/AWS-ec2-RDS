# AWS-EC2-RDS

## Project Overview

This project demonstrates the design and implementation of a highly available AWS web infrastructure using core AWS networking, compute, database, load balancing, content delivery, monitoring, and backup services.

The infrastructure is designed to distribute incoming web traffic across EC2 instances, scale compute capacity based on demand, store database information in Amazon RDS, and deliver website content through Amazon CloudFront.

The primary focus of this project is **AWS infrastructure implementation, configuration, security, monitoring, availability, and troubleshooting** rather than application development.

## Project Objectives

* Build a custom Amazon VPC environment.
* Configure public and private subnets.
* Deploy Ubuntu EC2 instances running Apache.
* Configure an Application Load Balancer to distribute traffic.
* Implement EC2 Auto Scaling based on CPU utilization.
* Deploy an Amazon RDS MySQL database.
* Configure secure connectivity between EC2 and RDS.
* Configure Amazon CloudFront with the load balancer as its origin.
* Monitor infrastructure using Amazon CloudWatch.
* Understand RDS automated backups and manual snapshots.
* Document implementation steps, testing results, and troubleshooting.

## AWS Services Used

| AWS Service                  | Purpose                                                       |
| ---------------------------- | ------------------------------------------------------------- |
| Amazon VPC                   | Provides an isolated virtual network                          |
| Public and private subnets   | Organize resources according to network accessibility         |
| Security Groups              | Control inbound and outbound traffic                          |
| Amazon EC2                   | Hosts the Apache web server                                   |
| Application Load Balancer    | Distributes incoming HTTP/HTTPS requests                      |
| EC2 Auto Scaling             | Adjusts the number of EC2 instances according to demand       |
| Amazon RDS for MySQL         | Provides a managed relational database                        |
| Amazon CloudFront            | Delivers content through a content delivery network           |
| Amazon CloudWatch            | Monitors metrics, alarms, and infrastructure activity         |
| Amazon RDS automated backups | Supports database recovery                                    |
| Amazon RDS snapshots         | Provides point-in-time database copies for recovery scenarios |

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
               ---------------------
               |                   |
               v                   v
          EC2 Instance 1       EC2 Instance 2
          Apache Web Server    Apache Web Server
               |                   |
               ---------+-----------
                         |
                         v
                  Amazon RDS MySQL
                  Private Database
```

Amazon CloudWatch monitors supported infrastructure metrics and alarms. EC2 Auto Scaling manages the number of EC2 instances. RDS automated backups and manual snapshots support database recovery.

**Note:** This diagram represents the intended architecture. Confirm the actual subnet placement, connectivity, target health, and service status against the deployed AWS resources before presenting the project as fully operational.

## Implementation Phases

### Phase 1: Networking

* Create or configure the VPC.
* Configure public and private subnets.
* Configure route tables and internet access where required.
* Configure security groups with restricted access.

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
* Verify that the registered targets are healthy.
* Test the website through the load balancer DNS name.

### Phase 4: Auto Scaling

* Create a launch template.
* Configure an Auto Scaling group.
* Attach the group to the target group.
* Configure minimum, desired, and maximum instance capacity.
* Configure a target tracking scaling policy with a 50% average CPU utilization target.
* Verify scaling activity when suitable load and metric conditions are met.

### Phase 5: Amazon RDS

* Deploy an RDS MySQL database.
* Configure the database in the intended VPC and subnet arrangement.
* Restrict database access through security groups.
* Configure the EC2 security group as the source of permitted MySQL traffic.
* Test connectivity from EC2 to RDS.
* Run basic SQL commands after successfully connecting.

### Phase 6: CloudFront

* Create a CloudFront distribution.
* Configure the Application Load Balancer as the origin.
* Configure the required cache behavior.
* Wait until the distribution is deployed.
* Test the website through the CloudFront distribution domain.
* Verify the behavior of dynamic content before enabling aggressive caching.

### Phase 7: Monitoring and Backups

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
| CloudFront              | Website is accessible through the distribution domain | Pending verification |
| CloudWatch              | Relevant metrics and alarms are visible               | Pending verification |
| RDS backups             | Backup configuration is verified                      | Pending verification |

Only mark a test as completed after collecting evidence that demonstrates the expected result.

## Security Considerations

* Avoid exposing the database directly to the public internet.
* Allow MySQL traffic on TCP port 3306 only from the required EC2 security group.
* Restrict SSH access to trusted IP addresses.
* Follow the principle of least privilege when assigning IAM permissions.
* Avoid storing database passwords or private keys in GitHub.
* Use HTTPS where configured and appropriate.
* Review security group rules and network access controls.

## Repository Structure

```text
aws-high-availability-infrastructure/
├── README.md
├── architecture/
│   └── aws-architecture.png
├── documentation/
│   ├── implementation.md
│   ├── testing.md
│   └── troubleshooting.md
└── screenshots/
    ├── networking/
    ├── ec2-apache/
    ├── load-balancer/
    ├── auto-scaling/
    ├── rds/
    ├── database-connection/
    ├── cloudfront/
    ├── cloudwatch/
    └── backups/
```

## Evidence and Screenshots

The repository should include screenshots demonstrating:

* VPC and subnet configuration.
* EC2 instance details and Apache service status.
* Load balancer listener and healthy target group.
* Auto Scaling group and scaling policy.
* RDS configuration and database security group.
* Successful EC2-to-RDS connection and SQL query output.
* CloudFront origin and deployed distribution.
* CloudWatch metrics and alarms.
* RDS automated backup settings and manual snapshot, if created.

Screenshots should demonstrate actual configuration and test results. Do not include passwords, access keys, private keys, or other sensitive information.

## Skills Demonstrated

* AWS infrastructure deployment
* VPC and subnet configuration
* Security group configuration
* EC2 and Apache administration
* Application Load Balancer configuration
* EC2 Auto Scaling
* Amazon RDS and MySQL connectivity
* CloudFront configuration
* CloudWatch monitoring
* Database backup concepts
* Linux command-line administration
* Infrastructure troubleshooting
* Technical documentation using GitHub

## Cost Management

AWS resources may incur charges even when they are used for learning.

* Stop or terminate resources when they are no longer required.
* Review EC2, load balancer, RDS, CloudFront, storage, and data transfer charges.
* Remove unused snapshots and other resources when they are no longer needed.
* Review the applicable AWS pricing and free-tier or credit eligibility before deploying resources.

## Conclusion

This project is intended to demonstrate practical knowledge of designing, deploying, monitoring, and troubleshooting AWS infrastructure using managed cloud services.

It emphasizes infrastructure administration, availability, scaling, database connectivity, security, content delivery, and operational documentation.

**Project status:** Update this section as the implementation and testing phases are completed.
