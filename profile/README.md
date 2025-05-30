# Drama Streaming MSA Project 

## 1. Project Overview

MSA-based API Bottleneck Tracing and Visualization Infrastructure

### Topic & Background 
This project was initiated with the question: "How much do API response delays impact user experience and revenue?" Inspired by Amazon’s report that a 100ms delay could reduce revenue by 1%, and Google’s discovery that a 0.5s delay leads to a 20% traffic drop, we focused on making API delays observable.

Our goal was to build an infrastructure that **traces inter-service traffic flow**, identifies bottlenecks, and **visualizes the delay in real-tim**e. We designed a microservice architecture (**MSA**) with **Spring Boot** and implemented observability using **Istio**, **Jaeger**, **Prometheus**, and **Grafana**.

---
## 2. Installation & Execution
### Requirements
- Java 17
- SpringBoot 3.2.5
- Docker & Kubernetes (Minikube or AWS EKS)
- Node.js 18
- React 19.1.0
- Node.js 22.16.0

### Setup
```
# Docker build
$ docker build -t drama-service:latest ./drama-service
$ docker build -t episode-service:latest ./episode-service
$ docker build -t stream-service:latest ./stream-service

# Terraform
$ terraform apply --auto-approve
```
---
## 3. Usage
### Scenario
1. When a user clicks a drama on the frontend, the following API sequence is triggered:
2. drama-service receives the initial request
3. It requests episode info from episode-service
4. Then calls stream-service to get a presigned S3 URL
5. This entire sequence is traced through Envoy proxies and visualized in Jaeger

```
GET /dramas         # List all dramas with episodes and stream URLs
GET /dramas/{id}    # Fetch detailed info about a single drama
```
---
## 4. Architecture and Observability
### Infrastructure Provisioning Workflow
1. Terraform provisions core AWS infrastructure components: VPC, subnets, IAM roles, EKS cluster, and the **OIDC** provider for service identity.
2. IAM Roles are securely linked to Kubernetes ServiceAccounts using **IRSA** (IAM Roles for Service Accounts), enabling Pod-level access control to AWS resources such as S3 or Load Balancers.
3. Helm is used to install key observability components into the Kubernetes cluster:
    - AWS ALB Controller for ingress traffic management
    - Istio for service mesh and traffic routing
    - Jaeger for distributed tracing
    - Prometheus and Grafana for metric collection and real-time dashboard visualization
4. Application Services (drama, episode, stream) are deployed as independent **Helm charts** and managed within the service mesh.

- Istio Gateway routes traffic from AWS ALB to Kubernetes
- VirtualService rules distribute traffic to drama → episode → stream services
- Each Pod has an **Envoy sidecar** to collect trace data
- Traces are sent to **Jaeger**
- Metrics are collected via **Prometheus** and visualized in Grafana
- Infrastructure is provisioned via Terraform, deployed via Helm and **GitHub Actions**

### IRSA (IAM Roles for Service Accounts)

We implemented IRSA to grant AWS permissions (like ALB or S3 access) to specific Kubernetes ServiceAccounts. OIDC providers were linked via Terraform, and Helm charts assigned IAM roles to services securely.

---
## 5. Architecture Diagram
![image](https://github.com/user-attachments/assets/d27dc057-a1f4-4711-a132-5a0475f0172d)

---
## 6. Team Member
| Hyomin An | HyeJin Shim | Yongbeom Kim | Chanhoon Kim | Juseung Lee |
| :---: | :---: | :---: | :---: | :---: |
| Team Leader, BE | Infra (Terraform, Helm) | CI/CD (Github Actions) | Infra (Terraform, Helm) | FE |
| <img src="https://avatars.githubusercontent.com/u/98948416?v=4" alt="hyomin profile" width="180" height="180"> | <img src="https://avatars.githubusercontent.com/u/204316388?v=4" alt="hyejin profile" width="180" height="180"> | <img src="https://avatars.githubusercontent.com/u/122729196?v=4" alt="yongbeom profile" width="180" height="180"> | <img src="https://avatars.githubusercontent.com/u/134241229?v=4" alt="chanhk1 profile" width="180" height="180"> | <img src="https://avatars.githubusercontent.com/u/194181560?v=4" alt="juseung profile" width="180" height="180">
