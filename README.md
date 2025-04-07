# 🚀 Todo ECS Blue-Green Deployment

This repository contains an **AWS CloudFormation template** for deploying a containerized **"Todo" application** using **Amazon ECS with Blue-Green Deployment** in the `eu-central-1` region.

> It provisions a **highly available and scalable infrastructure**, including VPC, subnets, ECS Fargate service, Application Load Balancer (ALB), and blue-green deployment using AWS CodeDeploy.

---

## 📌 Overview

The `template.yaml` provisions the following AWS resources:

- **VPC**: With public and private subnets across two Availability Zones (`eu-central-1a`, `eu-central-1b`).
- **Public Subnets**: Host the **ALB** for external access.
- **Private Subnets**: Host the **ECS Fargate tasks**.
- **NAT Gateway**: Enables outbound internet access from private subnets.
- **ECS Cluster**: Runs the containerized **"Todo"** application.
- **ALB**: Publicly accessible load balancer for traffic distribution.
- **Blue-Green Deployment**: Managed by **AWS CodeDeploy** for zero-downtime deployments.
- **Auto Scaling**: Scales the ECS service based on CPU utilization.

> 📝 *Note: This template does **not** provision an RDS database or S3 bucket. It focuses solely on ECS deployment.*

---

## ✅ Prerequisites

Before deploying:

- 🔧 [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) installed and configured
- 🐳 Container image pushed to your **Amazon ECR**
- 🔐 IAM permissions to create the required resources
- 🌍 Access to the `eu-central-1` region

---

## 🚀 Deployment Instructions

### 1. Clone the Repository

```bash
git clone <repository-url>
cd <repository-directory>
```

### 2. Update the Template

Replace the image URL in the ECS Task Definition:

```yaml
Image: <account-id>.dkr.ecr.eu-central-1.amazonaws.com/todo-repo:latest
```

---

### 3. (Optional) Validate the Template

```bash
aws cloudformation validate-template --template-body file://template.yaml
```

---

### 4. Deploy the Stack

```bash
aws cloudformation create-stack \
  --stack-name todo-ecs-stack \
  --template-body file://template.yaml \
  --region eu-central-1 \
  --capabilities CAPABILITY_NAMED_IAM
```

> ⚠️ Replace `todo-ecs-stack` with your preferred stack name.  
> The `CAPABILITY_NAMED_IAM` flag is required for IAM role creation.

---

### 5. Monitor Stack Creation

```bash
aws cloudformation describe-stacks \
  --stack-name todo-ecs-stack \
  --region eu-central-1
```

---

### 6. Access the Application

Retrieve the ALB URL:

```bash
aws cloudformation describe-stacks \
  --stack-name todo-ecs-stack \
  --region eu-central-1 \
  --query "Stacks[0].Outputs[?OutputKey=='TodoLoadBalancerURL'].OutputValue" \
  --output text
```

> Open the URL in your browser to access the deployed **"Todo"** app.

---

### 7. Perform a Blue-Green Deployment

- Push a new container image to your ECR repo.
- Trigger a deployment using **AWS CodeDeploy**:
    - Application name: `todo-application`
    - Deployment group: `todo-deployment-group`

---

## 🏠 Infrastructure Details

| Component            | Configuration |
|----------------------|---------------|
| **VPC**              | `10.0.0.0/16` |
| **Public Subnets**   | `10.0.1.0/24` (AZ1), `10.0.2.0/24` (AZ2) |
| **Private Subnets**  | `10.0.3.0/24` (AZ1), `10.0.4.0/24` (AZ2) |
| **ECS Task**         | `2 vCPU`, `4 GiB`, port `8080` |
| **ALB Listeners**    | Port `80` (blue), `8081` (green) |
| **Auto Scaling**     | 50% CPU target, min `1`, max `3` tasks |
| **Logging**          | `CloudWatch Logs Group: todo-log-group` |

---

## 📤 Outputs

After deployment, these values are exported:

| Output Key             | Description |
|------------------------|-------------|
| `TodoLoadBalancerDNS`  | DNS name of the ALB |
| `TodoLoadBalancerURL`  | HTTP URL to access the app |
| `CodeDeployAppName`    | Name of the CodeDeploy app (`todo-application`) |
| `CodeDeployGroupName`  | Deployment group name (`todo-deployment-group`) |

---

## 🧹 Cleanup

To delete the stack and all related resources:

```bash
aws cloudformation delete-stack --stack-name todo-ecs-stack --region eu-central-1
```

---

## ⚠️ Notes

- Ensure the **container image is uploaded** to ECR **before deployment**.
- The **NAT Gateway incurs costs** even when idle.
- Update the `HealthCheckPath` in the ALB if your app uses a custom health endpoint.

---

