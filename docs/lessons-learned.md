# Lessons Learned

This project reinforced several practical AWS infrastructure concepts that became clearer through hands-on deployment and troubleshooting.

## 1. A Public Subnet Is Defined by Routing

A subnet is not public because of its name.

It becomes public when resources in that subnet have a route to an Internet Gateway and the necessary public addressing and security controls.

This project made the relationship between subnets, route tables, Internet Gateways, and public IP addressing much more concrete.

---

## 2. Network Segmentation Should Follow Resource Requirements

Not every resource needs internet exposure.

The EC2 web tier requires public HTTP access, while the RDS database only needs connectivity from the application tier.

Separating these resources into public and private subnet tiers creates a cleaner and more secure architecture.

---

## 3. Security Groups Are More Flexible Than Static IP Rules

Using `web-server-sg` as the source for the RDS MySQL rule created a trust relationship between infrastructure roles rather than individual IP addresses.

This is easier to maintain and scale than hard-coding a specific EC2 private IP.

---

## 4. AWS Service Choices Affect Network Design

The original RDS deployment exposed an important lesson: managed AWS services can impose subnet and Availability Zone requirements.

The RDS deployment model directly affected the required DB subnet architecture.

Infrastructure should therefore be designed with the requirements of the services it will host in mind.

---

## 5. Resource Status Does Not Prove End-to-End Functionality

An EC2 instance showing `Running` or an RDS database showing `Available` only proves that AWS successfully provisioned the resource.

The complete application path still needs to be validated.

For this project, validation included:

* Accessing Apache through the EC2 public endpoint
* Connecting from EC2 to the private RDS endpoint
* Authenticating to MySQL
* Creating a database and table
* Writing and querying test records
* Confirming CloudWatch Agent operation

---

## 6. Error Messages Help Identify the Failing Layer

The RDS authentication error demonstrated the difference between a network failure and an authentication failure.

A timeout would have suggested a routing, Security Group, or service availability issue.

An `Access denied` response showed that the database had already been reached successfully and that the investigation should focus on credentials.

This reinforced the value of troubleshooting based on evidence rather than changing unrelated infrastructure.

---

## 7. Operating-System Accounts and Application Accounts Are Separate

The Linux `ec2-user` identity was initially confused with the MySQL database account.

The EC2 operating system and the RDS database maintain separate authentication systems.

Understanding these boundaries is important when troubleshooting systems made up of several infrastructure layers.

---

## 8. Generated Configuration Should Still Be Reviewed

The CloudWatch Agent configuration wizard initially created configuration that referenced an unnecessary collectd dependency.

This demonstrated that configuration generators and wizards can simplify deployment but should not replace understanding of the resulting configuration.

A smaller custom configuration was ultimately easier to understand and maintain.

---

## 9. Validate Configuration Before Changing Infrastructure

When the CloudWatch Agent reported a JSON translation error, the configuration was checked independently using:

```bash
python3 -m json.tool /opt/aws/amazon-cloudwatch-agent/etc/cloudwatch-agent.json
```

This identified the syntax problem without changing IAM permissions, networking, or reinstalling the agent.

The experience reinforced the practice of isolating the failing component before making changes.

---

## 10. IAM Roles Are Preferable to Stored Access Keys

The EC2 server required permission to publish CloudWatch Agent metrics.

Instead of storing AWS access keys on the server, an IAM role was attached to the instance.

This provides temporary AWS credentials and reduces the risk associated with long-lived secrets.

---

## 11. Monitoring Is Part of Infrastructure Deployment

Deploying a server is only part of operating it.

Adding CloudWatch metrics and alarms made the environment observable and demonstrated that infrastructure should be monitored after deployment rather than treated as complete once it is running.

---

## 12. Architecture Documentation Improves Understanding

Creating the architecture diagram required every resource and connection to be explained visually.

That process exposed unclear assumptions and made it easier to understand:

* which services belong inside the VPC
* which resources are public or private
* how traffic flows between tiers
* how IAM and CloudWatch relate to EC2
* why multiple Availability Zones were used

Documentation became part of the engineering process rather than something added only at the end.

---

# Overall Takeaway

The most valuable part of the project was not deploying individual AWS services.

It was understanding how networking, compute, databases, IAM, monitoring, and security controls interact as one system.

The project also reinforced a troubleshooting approach based on:

1. Reading the exact error
2. Identifying the infrastructure layer involved
3. Verifying known-good components
4. Changing one variable at a time
5. Validating the final result end-to-end

These principles are applicable beyond AWS and form a foundation for future infrastructure and cloud engineering work.
