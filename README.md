# Migration Plan to Modernize AWS Infrastructure

Migrating from a monolithic architecture to a modern, highly-available infrastructure is a significant undertaking that requires careful planning and execution. With such a significant undertaking, it's important that all members are involved in the process in different degrees.

Actively participatation of the developmet team in architectural decisions is crucial in undertaking a wholesome, empowering and efficient process.

The objective is to design and implement a migration plan to move from a monolithic AWS infrastructure to your improved architecture.

We first begin with a simple task of producing efficient minimal container images.

## Task 1: Quick Optimisation Challenge

```
FROM rust:1.81

# Install project dependencies (note this layer gets re-run on every change)
COPY ./Cargo.toml ./Cargo.toml
RUN cargo fetch

COPY ./src ./src

RUN cargo build --release
CMD ["./target/release/myapp"]
```

Optimisation
- As this is a Rust application, the latest stable Rust image was used. The reason this image was used rather than the slim, or alpine version is because it already has curl installed and will not require any package installation or updating reducing the process time. Build-essential package contains many essentials dependencies packages which won't be needed in this case.

You can find the list of available packages using the following commands:
- https://stackoverflow.com/questions/57803595/how-do-i-list-all-applications-that-are-contained-in-a-docker-container 

## Task 2: Improve The Architecture

This is where we focus on moving from a Monolithic architecture to a modern, highly-available infrastructure. The task will be broken down into a number of steps which I feel are important. The order of the steps can be flexible.

### Objective:
Design and implement a migration plan to transition the existing AWS infrastructure to a more advanced, highly available architecture.

This project assumes the current infrastructure is as follows.

### Current Infrastructure
- VPC with two public subnets across different availability zones
- EC2 instances running the applications
- Elastic Load Balancer for traffic distribution
- Security Group allowing HTTP traffic
- Simple deployment using user data script on EC2 instances

### Proposed Architecture Improvements

There are two main architectural computational choices available in AWS which meet the requirements.
- Microservics Arhitecture: where we break down the monolithic into smaller, indepedent services.
- Severlesss Computation Arhitecture: Leveraging the use of AWS Managed servicse like Lambda functions, API Gateways etc that can be event-driven.

In large scale project as this, there is no one simple answer, and the right tool whether microserves or serverless will depend on each scaenario. All services mentioned below work for both microservices and serverless architecture unless stated otherwise.

1. VPC and Networking improvements
- Private and Public Subnets: Divide the VPC into private and public subnets across multiple Availability Zones (AZs).
- NAT Gateways: Enable instances in private subnets to access the internet securely whilst preventing inbound traffic.
- Route Tables: Configure custom route tables for network traffic control.

2. High Availability and Scalability
- Multi AZ Deployment: Distribute resources across at least mulitple AZs for fault tolerance.
- Auto Scaling Groups (ASGs): Automatically adjust the number of EC2 instances based on load/demand
- Application Load Balancer (ALB): Replace ELB with ALB for advanced routing features.

3. Refactor Applications
- Identify boundaries: Refactor applications based on logical boundaries.
- Identify value: Categories services which add value to the business.

4. Containerise and Orchestrate (microservice)
- Containerise applications: Containerise application for consistency and scalability across enviromnents.
- Elastic Kubernetes Servics: Make use of EKS for container orchestration due to it's flexibility, support and vast capabilities.

5. Serverlesss Computation (serverless)
- AWS Lambda: migrate resources that are event-driven and have a run-time of less than 15 minutes without the need of state managenet
- Step functions: If state management is required, use step functions alongside AWS Lambda
- API Gateway: To expose AWS API's e.g. AWS Lambda via RESTful API

6. Infrastructure as Code (IaC)
- Terraform / Terragrunt: Import all manually deployed resources to terraform state and create all new resources using terraform.
- Version Control: Store IaC manifest in a Git repository.
- Lock Control: Store terraform state lock in DynamoDB
- Modular templates: Create or make use of reusable terraform modules for common infrastructure resources.

7. Continous Integration and Contineous Deployment (CI/CD)
- Pipleline: Setup the pipeline using: Github Actions OR CircleCI for CI and ArgoCD for CD.
- Testing: Run through application unit testing and infrastructure linting. 
- Blue/green deployment: consider how you can implement blue/green deployment to minimuse downtime during deployment
- Backup: Policies and processses for Disaster Recovery.

8. Security
- IAM Roles and Policies: Fine-grained access control for resources and IAM identities.
- Security Groups and Network ACL (firewall): limit access to inbound and outbound traffic rules in resources
- AWS Web Application Firewall (WAF): Add WAF after load balancer to protect against commonly uses web exploitations.
- Encryption: Encrypt resourcse at rest (KMS, S3, RDS, EC2) and during transit (HTTPS)

9. Cost Optimisation
- VM cost plan: buy Reserved Instances (RI) for resources we know we will need and/or Saving Plans to commit to spending $$ on resources for t time.
- Right-sizing resources: Regularly analyise and adjust resource size
- Create Prometheus and Graphana dashboard for visualising resource utilisaton.

### Migration Procedure

Stage 1: Planning and Preparation
1. Team Alignment
- Arrange meetings with all stackholders align on the goals, gather requirements and concerns.
- Demonstrate what success looks like.

2. Workshops
- Evaluate teams level of understanding in IaC, containerisation and EKS
- Conduct training sessions (inidivuals), workshops (groups) and documentations.

3. Resource Inventory
- Catelog all existing AWS resources and their configurations
- Catelog resource dependenceies and data flows between services.

Phase 2: Infrastructure setup
1. Implement IaC
- Choose tool based on teams skillset, in this example it's terraform
- Set up Git repository, S3 bucket for state control, and DynamoDB for state lock
- Structure the networking: VPC, Subnets, route teables, security groups.

2. Network Configuration
- Structure the networking: VPC, Subnets, route teables, security groups.
- Create and configure NAT Gateway, Internet Gateway and, Transit Gateway for multi-account setup.

3. Security
- Create separate IAM roles for developers, engineers, billing stackholders and admin
- Ensure resources are encrypted at rest using KMS and in transit
- Implement SSO for authentication into the infrastructure by users
- Consider 3rd party monitoring tools including a volunerability analysis of container images.
- Restrict how permissions can be allocated to users. Team lead or managers can request access to individuals.

Phase 3: Migrating applications
1. Containerisation
- Containerise applications using Docker
- Store the container images in Amazon Elastic Container Rigestry (ECR)
- Consider a single repository containing all images for a single source of truth.

2. Deploy EKS
- Create EKS using choosen IaC i.e. terraform
- Create necessary plugins for EKS to interact with terraform

4. Configure access to the public internet
- Deploy an Application load balancer in the public subnet
- Ensure there is access to the internet and from the internet - correctly configure listeners and target groups

5. Validate database during application migration
- Ensure the applications running in EKS are able to reach the database
- Ensure database credentials and securely stored in AWS secret manager.



For more detailed reading, please use the following resources:
- (Moving fom Monolithic to Microservice Architecture)[https://aws.amazon.com/compare/the-difference-between-monolithic-and-microservices-architecture/]