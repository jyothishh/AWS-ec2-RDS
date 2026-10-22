# AWS Infrastructure Project — Implementation Guide

## 1. Introduction

This document describes the implementation of an AWS infrastructure project using Amazon VPC, EC2, Application Load Balancer, EC2 Auto Scaling, Amazon RDS for MySQL, Amazon CloudFront, and Amazon CloudWatch.

The goal is to deploy a web server environment with load balancing, scaling, managed database services, content delivery, monitoring, and backup capabilities.

This document should be updated with the actual resource names, configurations, screenshots, and test results from the AWS environment.

## 2. Prerequisites

The following are required:

* An AWS account with permission to create the required resources.
* Access to the AWS Management Console.
* Basic Linux command-line knowledge.
* Basic understanding of networking and security groups.
* An Ubuntu EC2 instance for the web server.
* An RDS MySQL database.
* A GitHub repository for documentation and evidence.

Before beginning, review the pricing of the AWS resources you intend to use.

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
7. Review the network ACLs if connectivity problems occur.

The web-facing components must have the network access required for their intended role. The database should remain private and should not be made publicly accessible merely to resolve connectivity problems.

### Security Group Considerations

Configure security groups according to the traffic flow:

* CloudFront to the origin: permit the origin traffic required by the chosen design.
* Load balancer to EC2: permit the web traffic received from the load balancer security group.
* EC2 administration: restrict SSH access to trusted IP addresses.
* EC2 to RDS: permit MySQL traffic on TCP port 3306 from the EC2 security group.
* RDS inbound traffic: do not allow unrestricted access from the public internet.

### Evidence to Collect

Save screenshots showing:

* VPC details.
* Public and private subnet details.
* Route tables.
* EC2 and RDS security group rules.

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

Distribute incoming web traffic across the registered EC2 targets.

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

A target of 50% CPU utilization is a scaling target, not a guarantee that a new instance launches immediately whenever CPU exceeds 50%. Scaling behavior depends on metrics, alarm evaluation, cooldown or warm-up behavior, and capacity limits.

### Evidence to Collect

* Launch template details.
* Auto Scaling group configuration.
* Minimum, desired, and maximum capacity.
* Scaling policy.
* CloudWatch alarm details.
* Scaling activity history.
* Newly launched instance and its target health, if scaling is successfully tested.

Recommended folder: `screenshots/auto-scaling/`

## 7. Amazon RDS for MySQL

### Objective

Deploy a managed MySQL database and allow authorized application or administration traffic from the EC2 environment.

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
getent hosts db-ecw.czy6ck0g6em5.ap-southeast-2.rds.amazonaws.com
```

The endpoint should resolve to an address reachable from the EC2 network.

#### Step 2: Test TCP Connectivity

Run:

```bash
nc -vz -w 5 db-ecw.czy6ck0g6em5.ap-southeast-2.rds.amazonaws.com 3306
```

If the connection times out, investigate the security group attached to RDS first. Confirm that its inbound rule allows TCP port 3306 from the EC2 security group.

Also check the EC2 outbound rules and, if necessary, the subnet route tables and network ACLs.

Do not treat a timeout as a password problem. Password and authentication issues can be investigated after the network connection succeeds.

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
mysql -h db-ecw.czy6ck0g6em5.ap-southeast-2.rds.amazonaws.com -P 3306 -u admin -p
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

Only create resources that are appropriate for your database and permissions. Clean up the test database if it is no longer needed.

### Evidence to Collect

* Successful TCP connectivity test.
* Successful MySQL connection.
* SQL query output.
* Relevant error output if a test fails, with sensitive information removed.

Recommended folder: `screenshots/database-connection/`

**Implementation status:** Do not mark this section complete until the TCP connection and MySQL session have been successfully verified.

## 9. Amazon CloudFront

### Objective

Configure CloudFront to deliver web content through a content delivery network, using the Application Load Balancer as the origin.

### Implementation Steps

1. Open the Amazon CloudFront console.
2. Create a distribution.
3. Configure the Application Load Balancer as the origin.
4. Select the correct origin protocol policy.
5. Configure the viewer protocol policy, such as redirecting HTTP requests to HTTPS.
6. Review the cache policy.
7. Use suitable caching behavior for the website's content.
8. Avoid aggressive caching for dynamic content unless it has been designed and tested for caching.
9. Create the distribution.
10. Wait until the distribution status indicates it is deployed.
11. Test the website using the CloudFront distribution domain.
12. Verify that the response is correct and that the origin is reachable.

For an origin that uses HTTPS, the load balancer must have a suitable certificate and HTTPS listener. Do not configure HTTPS to the origin without verifying the origin-side TLS setup.

CloudFront does not replace the load balancer or repair unhealthy EC2 targets. Confirm that the origin works before troubleshooting CloudFront.

### Evidence to Collect

* CloudFront distribution details.
* Origin configuration showing the load balancer.
* Cache and viewer protocol policies.
* Distribution status showing deployed.
* Successful website response through the CloudFront domain.

Recommended folder: `screenshots/cloudfront/`

## 10. Amazon CloudWatch Monitoring

### Objective

Monitor infrastructure performance and observe events that affect scaling and availability.

### Implementation Steps

1. Open the CloudWatch console.
2. Review EC2 CPU utilization.
3. Review the load balancer metrics.
4. Review target group health-related metrics.
5. Review RDS metrics such as CPU utilization and database connections.
6. Inspect the alarms associated with Auto Scaling.
7. Configure additional alarms if required.
8. Record the alarm thresholds and evaluation settings.
9. If testing scaling, observe the metric and the resulting Auto Scaling activity.

CloudWatch metrics and alarms can help identify performance problems, unhealthy targets, and scaling events. They do not guarantee that an issue is automatically resolved.

### Evidence to Collect

* EC2 CPU utilization graph.
* Load balancer metrics.
* RDS metrics.
* Scaling-related alarms.
* Auto Scaling activity history, if applicable.

Recommended folder: `screenshots/cloudwatch/`

## 11. RDS Backups and Snapshots

### Objective

Understand how database backups support recovery.

### Implementation Steps

1. Open the RDS console.
2. Select the database.
3. Review its automated backup retention settings.
4. Record the backup window and retention period.
5. Review the available automated backups.
6. If required, create a manual DB snapshot.
7. Record the snapshot identifier and status.
8. Understand the process for restoring a snapshot to a new database instance.

Automated backups and manual snapshots serve related but different purposes. A manual snapshot remains available until it is deleted, while automated backup retention follows the configured retention period and AWS behavior.

Creating a snapshot alone does not prove that database recovery works. A restore test provides stronger evidence, but it can create additional costs.

### Evidence to Collect

* Automated backup configuration.
* Backup retention period.
* Snapshot details and status.
* Restore test evidence, if a restore test is performed.

Recommended folder: `screenshots/backups/`

## 12. Final Validation

Before considering the project complete, verify the following:

* [ ] VPC and subnet configuration is documented.
* [ ] Apache is active on the EC2 instance.
* [ ] The load balancer has healthy targets.
* [ ] The website is accessible through the load balancer.
* [ ] Auto Scaling settings are documented.
* [ ] The scaling policy and associated alarms are verified.
* [ ] RDS is available and remains appropriately private.
* [ ] EC2-to-RDS TCP connectivity succeeds.
* [ ] MySQL login and SQL queries succeed.
* [ ] CloudFront distribution is deployed.
* [ ] The website is accessible through CloudFront.
* [ ] CloudWatch metrics and alarms are documented.
* [ ] RDS backup settings are verified.
* [ ] Required screenshots are organized in GitHub.
* [ ] No passwords, access keys, or private keys are committed.

## 13. Troubleshooting and Lessons Learned

Record each issue using the following format:

### Issue

Describe the observed problem.

### Symptoms

Record the error message or unexpected behavior.

### Investigation

List the checks and commands performed.

### Root Cause

Document the cause only after it has been confirmed.

### Resolution

Record the configuration change or command that resolved the issue.

### Verification

Record the test that demonstrated the issue was resolved.

### Evidence

Include a screenshot or sanitized command output.

For example, if the EC2-to-RDS connection times out, record the TCP test, inspect the RDS security group, and verify that the inbound rule references the correct EC2 security group. Update the root cause and resolution only after connectivity has been restored and tested.

## 14. Conclusion

This implementation demonstrates practical experience with AWS networking, Linux web server administration, load balancing, Auto Scaling, managed databases, content delivery, monitoring, backups, and troubleshooting.

The final project report should distinguish between resources that were configured, features that were tested successfully, and features that remain incomplete.
