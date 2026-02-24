# AWS Well-Architected Framework & Cloud Adoption Framework Assessment
## Two-Tier Web Application Migration

**Author:** Senior Cloud Engineer  
**Date:** February 23, 2026  
**Project:** On-Premises to AWS Migration – Two-Tier Web Application

---

## Executive Summary

This document presents a comprehensive evaluation of a two-tier web application migration from on-premises infrastructure to AWS. The assessment applies the five pillars of the AWS Well-Architected Framework (WAF) and the six perspectives of the AWS Cloud Adoption Framework (CAF) to identify risks, recommend improvements, and propose a resilient, cost-effective target architecture.

---

## Task 1 – Existing Architecture Review

### Current Workload Components

The existing on-premises architecture consists of:

- **Frontend Tier:** A web server (e.g., Apache/Nginx) serving a monolithic or single-page application, deployed on a single physical or virtual machine.
- **Backend Tier:** A relational database server (e.g., MySQL/PostgreSQL) running on a separate on-premises machine.
- **Networking:** Direct connection between frontend and database over a flat internal network with no segmentation.
- **Load Balancing:** None – single point of ingress.
- **Storage:** Local disk storage for both application files and database data.
- **Backup:** No documented backup or disaster recovery strategy.
- **Security:** Broad firewall rules; likely open ports and no WAF or DDoS protection.
- **Monitoring:** Minimal – likely OS-level metrics only, no centralized logging.
- **Deployment:** Manual, no CI/CD pipeline.

### Identified Risks and Weaknesses

| Risk Area | Description |
|---|---|
| Single Point of Failure | No redundancy across any tier; one server failure brings down the entire application |
| No Backup Strategy | Data loss risk is critical; no RPO/RTO defined |
| No High Availability | Single-AZ equivalent deployment on-premises |
| Open Security Groups (ports) | Overly permissive network rules expose attack surface |
| No Scalability | Cannot handle traffic spikes; no auto-scaling mechanism |
| No Monitoring or Alerting | Incidents are discovered reactively, not proactively |
| Manual Deployments | Human error risk; no rollback capability |
| Unencrypted Data | No evidence of encryption in transit or at rest |

---

## Task 2 – AWS Well-Architected Framework Evaluation

### WAF Assessment Table

| Pillar | Observation | Improvement Recommendation | Supporting AWS Service |
|---|---|---|---|
| **Operational Excellence** | **Strength:** Team has existing operational runbooks for the on-premises workload. **Weakness:** No CI/CD pipeline; all deployments are manual, increasing human error risk and slowing release cycles. | Implement Infrastructure as Code (IaC) and automated deployment pipelines. Define operational metrics and automate responses to common events. | AWS CodePipeline, AWS CodeDeploy, AWS CloudFormation / CDK, Amazon CloudWatch |
| **Security** | **Strength:** The database is on a separate server, providing basic logical separation. **Weakness:** No encryption at rest or in transit; overly broad network rules; no identity-based access controls; no Web Application Firewall. | Apply least-privilege IAM roles, enable encryption at rest and in transit, restrict security groups to minimum required ports, and deploy a WAF in front of the public-facing tier. | AWS IAM, AWS WAF, AWS KMS, AWS Secrets Manager, Amazon VPC Security Groups & NACLs, AWS Certificate Manager (ACM) |
| **Reliability** | **Strength:** The two-tier design separates concerns between presentation and data layers. **Weakness:** Single instance deployment with no redundancy; no backup policy; no disaster recovery plan; no health checks or automatic failover. | Deploy across multiple Availability Zones with Auto Scaling. Enable automated database backups with point-in-time recovery and define RTO/RPO targets. | Elastic Load Balancing (ALB), Amazon EC2 Auto Scaling, Amazon RDS Multi-AZ, AWS Backup, Amazon Route 53 health checks |
| **Performance Efficiency** | **Strength:** The workload's compute and storage are currently sized for known peak load. **Weakness:** No caching layer; static assets served directly by the application server; no CDN; no ability to right-size or scale dynamically. | Introduce a CDN to cache static assets globally, add an in-memory caching layer for database query results, and use performance monitoring to continuously right-size resources. | Amazon CloudFront, Amazon ElastiCache (Redis/Memcached), AWS Compute Optimizer, Amazon RDS Read Replicas |
| **Cost Optimization** | **Strength:** On-premises CapEx model provides predictable costs. **Weakness:** Over-provisioned hardware sitting idle during off-peak hours; no mechanism for resource lifecycle management; no visibility into cloud spend. | Use auto-scaling to match compute capacity to actual demand, leverage Reserved Instances or Savings Plans for baseline workloads, and enable cost visibility dashboards. | AWS Cost Explorer, AWS Budgets, EC2 Auto Scaling, EC2 Reserved Instances / Savings Plans, AWS Trusted Advisor |

---

## Task 3 – AWS Cloud Adoption Framework (CAF) Analysis

### Business Perspective

The Business perspective ensures cloud investments accelerate digital transformation ambitions and business outcomes. For this migration, the primary business drivers are improved application availability, reduced operational overhead, and the ability to scale rapidly with demand without proportional capital expenditure.

**Readiness Assessment:** The organization has a clear migration intent, but business stakeholders lack visibility into cloud ROI. There is no formal cloud business case or defined KPIs tied to migration success.

**Key Actions:**
- Develop a cloud business case quantifying TCO savings versus on-premises.
- Define business KPIs: uptime SLA (e.g., 99.9%), mean time to recovery (MTTR), and deployment frequency.
- Align migration milestones with business value delivery checkpoints.
- Engage senior leadership to establish cloud as a strategic priority, not just an IT project.
- Identify new capabilities enabled by cloud (e.g., global reach, ML/AI, faster feature delivery) to justify investment beyond cost savings.

A well-defined business case, communicated clearly to leadership, will ensure continued sponsorship and resource allocation throughout the migration journey.

---

### People Perspective

The People perspective focuses on organizational change management, training, and culture to enable cloud adoption. A successful migration is as much a people challenge as a technical one.

**Readiness Assessment:** The existing team has strong on-premises infrastructure skills. However, there is a significant skills gap in AWS services, cloud-native design patterns, and DevOps practices. Without upskilling, the team risks replicating on-premises anti-patterns in the cloud (a "lift and shift" without optimization).

**Key Actions:**
- Conduct a skills gap assessment across engineering, operations, and security teams.
- Enroll team members in AWS training paths: AWS Cloud Practitioner, Solutions Architect Associate, and SysOps Administrator.
- Establish a cloud center of excellence (CCoE) or cloud champions program to drive internal knowledge sharing.
- Introduce DevOps and SRE culture: shared responsibility, blameless post-mortems, and continuous improvement.
- Create change management communications to reduce organizational resistance and set clear expectations around new ways of working.

Investing in people is the highest-leverage activity for a sustainable cloud adoption program.

---

### Governance Perspective

The Governance perspective ensures that cloud usage aligns with organizational policies, regulatory requirements, and risk management frameworks. Without governance guardrails, cloud sprawl, security gaps, and runaway costs quickly emerge.

**Readiness Assessment:** The organization currently lacks cloud governance policies, tagging standards, and a cloud financial management (FinOps) practice. Compliance requirements (e.g., data residency, audit logging) have not been mapped to AWS controls.

**Key Actions:**
- Define and enforce a resource tagging strategy (environment, owner, cost center, project) using AWS Config rules and Service Control Policies (SCPs) via AWS Organizations.
- Establish a cloud governance committee to review architecture decisions, spending, and security posture monthly.
- Map regulatory requirements to AWS compliance programs (e.g., SOC 2, ISO 27001, GDPR).
- Enable AWS CloudTrail for full API-level audit logging across all accounts.
- Set budget alerts using AWS Budgets and define escalation procedures for cost anomalies.
- Use AWS Control Tower to enforce governance baselines at the multi-account level.

Governance should be implemented from day one, as retrofitting controls after growth is exponentially more difficult.

---

### Platform Perspective

The Platform perspective covers the design, implementation, and ongoing optimization of the cloud environment and the services running within it. This is where architectural decisions become concrete.

**Readiness Assessment:** The current architecture is a straightforward two-tier system. The team has the baseline capability to provision infrastructure but lacks experience with managed AWS services, VPC design, and cloud-native architectural patterns such as auto-scaling, managed databases, and serverless components.

**Key Actions:**
- Design a multi-tier VPC with public subnets (ALB), private subnets (EC2 application servers), and isolated subnets (RDS database) following the principle of least privilege networking.
- Migrate from self-managed database servers to Amazon RDS with Multi-AZ for managed patching, backups, and failover.
- Adopt Infrastructure as Code from day one using AWS CloudFormation or CDK to ensure environments are reproducible and version-controlled.
- Implement a CI/CD pipeline using AWS CodePipeline and CodeDeploy to automate build, test, and deployment workflows.
- Establish environment parity: development, staging, and production environments should mirror each other in architecture to reduce deployment risk.

---

### Security Perspective

The Security perspective ensures that the organization achieves the confidentiality, integrity, and availability of its data and cloud workloads. In the cloud, security is a shared responsibility model.

**Readiness Assessment:** The current security posture is weak. There is no WAF, no centralized log management, no secret rotation, and no encryption strategy. The team is unaware of AWS's shared responsibility model, leading to potential misconfigurations.

**Key Actions:**
- Train all engineers on the AWS Shared Responsibility Model.
- Enable AWS Security Hub as the central dashboard for security posture across services.
- Deploy Amazon GuardDuty for continuous threat detection and anomaly monitoring.
- Enforce multi-factor authentication (MFA) on all IAM users; use IAM Identity Center (SSO) for federated access.
- Replace hardcoded credentials with AWS Secrets Manager for database passwords and API keys, with automatic rotation enabled.
- Encrypt all data at rest using AWS KMS and enforce TLS 1.2+ for all data in transit.
- Deploy AWS WAF in front of the Application Load Balancer to protect against OWASP Top 10 threats and DDoS attacks.

Security must be embedded into every phase of the migration, not bolted on afterward.

---

### Operations Perspective

The Operations perspective ensures the cloud environment is run and managed effectively, meeting agreed-upon business and service level requirements. This encompasses monitoring, incident response, and continuous improvement.

**Readiness Assessment:** The current operations model is reactive — incidents are discovered by users or through ad hoc checks. There are no automated alerts, runbooks are informal, and there is no defined incident response process. The move to AWS must transform operations to a proactive, automated model.

**Key Actions:**
- Implement centralized observability using Amazon CloudWatch for metrics, logs, and alarms; integrate with AWS X-Ray for distributed tracing.
- Define SLOs and SLAs for application availability and latency; build dashboards and alerts that track these in real time.
- Create documented operational runbooks for common failure scenarios (e.g., instance failure, database failover, high CPU alerts) and automate responses using AWS Systems Manager Automation.
- Establish a formal incident response process with defined severity levels, escalation paths, and post-incident review procedures.
- Use AWS Systems Manager Session Manager to eliminate the need for bastion hosts and SSH key management for operational access.
- Conduct regular chaos engineering exercises (using AWS Fault Injection Simulator) to validate resilience before incidents occur in production.

---

## Task 4 – Improved Architecture Design

### Revised AWS Architecture Description

The improved architecture adopts a multi-tier, highly available design deployed across two Availability Zones (AZs) in a single AWS Region. All five WAF pillars are addressed through specific design choices.

#### Network Design (VPC)

A custom VPC is segmented into three subnet tiers across two AZs:

- **Public Subnets:** Host the Application Load Balancer (ALB) and NAT Gateways. Internet-facing traffic enters here only.
- **Private Application Subnets:** EC2 instances in an Auto Scaling Group running the web application. No direct internet access; outbound traffic routed through NAT Gateway.
- **Private Database Subnets:** Amazon RDS (Multi-AZ) with no internet route. Access restricted to the application tier via Security Group rules.

#### Frontend Tier

- **Amazon CloudFront** CDN sits in front of the ALB, caching static assets at edge locations globally. This reduces latency for end users and decreases load on origin servers.
- **AWS WAF** is attached to CloudFront to inspect and block malicious traffic (SQL injection, XSS, rate limiting).
- **Application Load Balancer (ALB)** distributes traffic across EC2 instances in multiple AZs. ALB performs health checks and automatically removes unhealthy instances.

#### Application Tier

- **Amazon EC2 Auto Scaling Group** maintains a minimum of two instances (one per AZ) and scales based on CPU utilization and request count metrics from CloudWatch.
- **EC2 instances** run in private subnets. They use IAM Instance Profiles (not hardcoded credentials) to access AWS services.
- **AWS Systems Manager** enables patch management, remote session access (no SSH keys), and operational automation.

#### Database Tier

- **Amazon RDS (MySQL/PostgreSQL) with Multi-AZ** deployment provides synchronous replication to a standby in a second AZ. Automated failover occurs within 60–120 seconds.
- **Automated backups** are enabled with 7-day retention. Point-in-time recovery (PITR) supports fine-grained data restoration.
- **Amazon ElastiCache (Redis)** caches frequent database queries, reducing RDS load and improving response times.
- **AWS Secrets Manager** stores database credentials with automatic rotation.

#### Storage

- **Amazon S3** hosts static assets (images, CSS, JS) served via CloudFront. Versioning and lifecycle policies manage object retention.
- **Amazon EBS** gp3 volumes for EC2 instances with encryption enabled via KMS.

#### Security Controls

- IAM roles with least-privilege policies for all compute resources.
- Security Groups follow a tiered model: ALB → App tier → DB tier.
- NACLs provide subnet-level network filtering as an additional layer.
- AWS KMS encrypts EBS volumes, RDS instances, S3 objects, and Secrets Manager secrets.
- AWS Certificate Manager (ACM) provides free TLS certificates for ALB and CloudFront.
- Amazon GuardDuty monitors for threats; AWS Security Hub aggregates findings.
- AWS CloudTrail logs all API calls for audit compliance.

#### Monitoring & Operations

- **Amazon CloudWatch** dashboards track key metrics: ALB request count, error rates, EC2 CPU/memory, RDS connections, and ElastiCache hit ratio.
- **CloudWatch Alarms** trigger SNS notifications and Auto Scaling actions.
- **AWS X-Ray** provides distributed tracing for application performance debugging.
- **AWS Systems Manager Automation** enables runbook-driven incident response.

#### CI/CD Pipeline

- **AWS CodePipeline** orchestrates the deployment workflow: source (CodeCommit/GitHub) → build (CodeBuild) → test → deploy (CodeDeploy with Blue/Green deployments to the Auto Scaling Group).
- **AWS CloudFormation / CDK** manages all infrastructure as code with environment-specific parameter sets.

---

### Architecture Diagram Description

```
Internet
    │
    ▼
[Amazon CloudFront + AWS WAF]
    │
    ▼
[Application Load Balancer]  ← Public Subnet (AZ-1 & AZ-2)
    │
    ▼
[EC2 Auto Scaling Group]     ← Private App Subnet (AZ-1 & AZ-2)
[ElastiCache (Redis)]
    │
    ▼
[Amazon RDS Multi-AZ]        ← Private DB Subnet (AZ-1 & AZ-2)
    │ (Primary)         │ (Standby)
   AZ-1               AZ-2

Supporting Services:
- S3 (static assets) → CloudFront
- Secrets Manager (DB creds)
- KMS (encryption)
- CloudWatch (monitoring)
- CloudTrail (audit logs)
- GuardDuty (threat detection)
- CodePipeline (CI/CD)
- Systems Manager (ops)
```

---

## Reflection

Completing this assessment reinforced the importance of treating architecture as a continuous process rather than a one-time design exercise. The most significant insight was recognizing how deeply interconnected the five WAF pillars are — a weakness in Security creates operational burden, which in turn drives up costs and degrades reliability. Applying the CAF perspectives revealed that technical architecture, while critical, represents only a fraction of migration success. People, governance, and operational maturity are equally decisive factors that are often underinvested.

The exercise also highlighted the risk of "lift and shift" thinking. Simply rehosting on-premises servers to EC2 without adopting managed services like RDS Multi-AZ, Auto Scaling, and CloudFront replicates on-premises fragility in the cloud at cloud prices — the worst of both worlds. True cloud benefit requires deliberate design choices aligned with well-architected principles from the outset.

Finally, the CAF analysis demonstrated that governance and security must be established before workloads go live, not retrofitted after incidents occur. The investment in upfront design pays dividends in reduced toil, faster recovery times, and the confidence to innovate at pace.

---

*Document prepared in accordance with AWS Well-Architected Framework and AWS Cloud Adoption Framework v3.0 best practices.*
