# Moving to a more advanced, highly-available architecture.

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

This project assumes the current infrastructure is as follows.

### Current Infrastructure
- VPC with two public subnets across different availability zones
- EC2 instances running the applications
- Elastic Load Balancer for traffic distribution
- Security Group allowing HTTP traffic
- Simple deployment using user data script on EC2 instances
-


### 1. Understanding the current infrastructure

This is where it's important to have teams across the engineering devisions onboard. We must understand the existing infrastructure to identify components, dependencies, and areas of improvements.

- Components and dependencies: 
-- Categorise what applications are being run in the EC2
-- Deep dive and understand of the user data and why certain choices were made
-- Investigate why the a golden image with user data was not backed
-- Identify dependcies between applications running in the monolith

- Measuring performance
-- Set a benchmark on current resource utilisation, response time, and throughput
-- Set a benchmark on cost / month
-- Identify current bottlenecks and steps where scaling will be limited.

- Security review
-- Assess the current security measures
-- Categories current potential volunerabilities in the pubic subnet setup.

### 2. Assessing potential improved architecture

Here we assess the candidates for modern, scalable and secure architectures.



For more detailed reading, please use the following resources:
- (Moving fom Monolithic to Microservice Architecture)[https://aws.amazon.com/compare/the-difference-between-monolithic-and-microservices-architecture/]