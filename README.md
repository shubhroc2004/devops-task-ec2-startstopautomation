***

# **README.md**

## **EC2 Start/Stop Automation Using EventBridge, SSM Automation, and Terraform**

This project automates the start and stop lifecycle of an EC2 instance using AWS EventBridge, AWS Systems Manager (SSM) Automation, and Terraform (Infrastructure as Code). It also includes manual validation steps and CloudTrail/SSM evidence to demonstrate successful automation.

***

## **📌 Project Overview**

The solution is divided into three phases:

### **PHASE 1 — Manual EC2 Start/Stop (Console & CLI)**

Manually started and stopped the target EC2 instance using both the AWS Console and AWS CLI to validate IAM permissions and basic EC2 behavior.

### **PHASE 2 — EventBridge Automation (Console)**

Configured EventBridge scheduled rules that invoke SSM Automation documents (`AWS-StartEC2Instance` and `AWS-StopEC2Instance`) to automatically start and stop the EC2 instance based on cron schedules.

### **PHASE 3 — Terraform Automation (IaC Implementation)**

Implemented the entire automation (IAM Roles, Policies, EventBridge rules, SSM targets) using Terraform to make the solution fully reproducible and parameter‑driven.

***

## **✔ Acceptance Criteria**

### **Functional Requirements**

*   Manual EC2 start/stop works via Console & CLI
*   EventBridge scheduled rules trigger successfully
*   EC2 instance changes state at the correct scheduled time
*   Terraform provisions:
    *   IAM Roles
    *   EventBridge start rule
    *   EventBridge stop rule
    *   SSM Automation targets
*   `terraform apply` runs without errors
*   `terraform destroy` removes all resources cleanly

### **Automation Quality**

*   Terraform code is modular and reusable
*   Variables used for all dynamic values
*   No hard‑coded AWS Account IDs
*   IAM follows least‑privilege design

### **Validation**

*   Schedule time changes still trigger automation correctly
*   No manual steps required after Terraform deployment

***

# **PHASE 1 – Manual Operations**

### **Console**

*   Navigated to EC2 console and manually started/stopped the instance.
*   Verified status transitions between **Running → Stopped → Running**.

### **CLI**

*   Used CloudShell to:
    *   Launch a new instance
    *   Stop an instance
    *   Start an instance
*   Verified instance state changes in the console.

***

# **PHASE 2 – EventBridge Automation (Console)**

## **IAM Roles and Policies**

### **1. Automation Assume Role**

*   Permissions:
    *   `ec2:StartInstances`, `ec2:StopInstances` (write)
    *   `ec2:DescribeInstances` (list)
*   Trust policy allows `ssm.amazonaws.com`
*   Scoped only to the specific EC2 instance (least privilege)

### **2. EventBridge Execution Role**

*   Permissions:
    *   `ssm:StartAutomationExecution`
    *   `iam:PassRole` (only for the Automation Execution Role)
*   Trust policy:
    *   Principal: `events.amazonaws.com`
    *   Conditions:
        *   `aws:SourceAccount`
        *   `ArnLike` on `aws:SourceArn` for scheduled rule ARNs
*   Ensures only the correct EventBridge rules can assume it

***

## **EventBridge Scheduled Rules**

### **Start EC2 Rule**

*   Cron-based schedule triggers SSM Automation to start the instance.
*   Passes AutomationAssumeRole to SSM.

### **Stop EC2 Rule**

*   Cron-based schedule triggers SSM Automation to stop the instance.
*   Securely invokes SSM through the EventBridge Execution Role.

### **Rule Creation Summary**

*   Chose cron expressions for precise timing.
*   Selected SSM Automation documents as targets.
*   Passed parameters:
    *   Instance ID
    *   AutomationAssumeRole
*   Attached EventBridge Execution Role as the rule’s IAM role.

***

## **Automation Output (Console Validation)**

### **1. CloudTrail Event History**

Confirms scheduled rules triggered at the correct times and initiated `StartAutomationExecution`.

### **2. SSM Automation Execution History**

Shows both Automation documents executed successfully for start and stop actions.

### **3. EC2 Instance Status**

Instance transitions correctly:

*   **Running → Stopped** for stop rule
*   **Stopped → Running** for start rule

### **4. EventBridge Rule Monitoring**

Monitoring tab shows successful and timely invocations for both scheduled rules.

***

# **PHASE 3 – Terraform (IaC Implementation)**

## **Environment Setup**

*   Connected to an Amazon Linux EC2 instance.
*   Installed and configured AWS CLI.
*   Installed Terraform via HashiCorp repository.
*   Created a working directory for configuration files.

***

## **Terraform Files Overview**

### **main.tf**

Defines the AWS provider and serves as the entry point for Terraform configuration.

### **terraform.tf**

Specifies required provider versions and Terraform version constraints.

### **variables.tf & secrets.auto.tfvars**

Centralize reusable values:

*   Account ID
*   Instance ID
*   Region
*   Start/Stop SSM Automation documents

### **iam.tf**

Configures IAM roles and permissions for:

*   **SSM Automation Role**
*   **EventBridge Execution Role**

### **eventbridge\_ssm.tf**

*   Defines `aws_cloudwatch_event_rule.ec2_start_rule` and `aws_cloudwatch_event_rule.ec2_stop_rule`
*   Prepares SSM inputs (instance ID + AutomationAssumeRole)
*   Builds Automation document ARNs
*   Creates EventBridge targets for start/stop
*   Links them with the Execution Role via `role_arn`
*   Provides correct JSON input for SSM automation

***

## **Terraform Commands**

### **terraform init**

Initializes providers and creates the state file.

### **terraform validate & terraform fmt**

Ensures configuration correctness and formatting.

### **terraform plan**

Shows the IAM roles, policies, EventBridge schedules, and SSM Automation targets that will be created.

### **terraform apply**

Provisions the full automation workflow and connects all components.

### **terraform destroy**

Removes all created AWS resources cleanly.

***

# **Final Results**

*   Scheduled rules triggered Automation at the exact cron times
*   SSM Automation executed start/stop workflows correctly
*   EC2 instance state updated accordingly
*   Monitoring and CloudTrail logs confirmed successful execution
*   IAM roles enforced least‑privilege behavior
*   Terraform fully automated provisioning and teardown

***

# **Challenges & Resolutions**

*   Rate-based schedules caused workflow overlap → replaced with CRON
*   Incorrect ARN format (`*` instead of account ID) → corrected
*   Trust policy mismatch (`StringEquals` vs `ArnLike`) → fixed to allow wildcard ARNs
*   Minor Terraform syntax errors → resolved through validation and testing

***

# **Conclusion**

This project successfully demonstrates how to automate EC2 lifecycle management using EventBridge and SSM Automation, fully orchestrated through Terraform. The design follows AWS best practices, uses least‑privilege IAM roles, avoids hardcoded values, and ensures the entire system is reproducible and maintainable.

***
