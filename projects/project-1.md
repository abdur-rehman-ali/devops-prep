# Production-Ready 3-Tier Application — Requirements

## 1. Project Overview

Build, deploy, and operate a production-ready **3-tier web application** using:

* **Frontend:** Next.js
* **Backend/API:** Python + Django + Django REST Framework
* **Database:** PostgreSQL
* **Cloud:** AWS
* **Infrastructure as Code:** Terraform
* **CI/CD:** GitHub Actions
* **Load Balancing:** AWS Application Load Balancer
* **Compute:** Amazon EC2 with Auto Scaling
* **Database Hosting:** Amazon RDS for PostgreSQL
* **CDN:** Amazon CloudFront
* **Monitoring:** Amazon CloudWatch
* **Secrets:** AWS Secrets Manager or AWS Systems Manager Parameter Store

The primary objective is to demonstrate the ability to **build, deploy, secure, scale, monitor, update, back up, recover, and troubleshoot a production-style application**.

The application itself can remain simple. The primary focus of this project is **infrastructure, automation, reliability, security, and operations**.

---

# 2. Application Requirements

## 2.1 Frontend

The frontend must be implemented using **Next.js**.

Requirements:

* Provide a functional web interface.
* Communicate with the Django backend through HTTP/HTTPS APIs.
* Support production builds.
* Run as an independently deployable application.
* Obtain environment-specific configuration at runtime or deployment time.
* Must not contain hard-coded production secrets.
* Must expose a health-check mechanism suitable for infrastructure monitoring.

The frontend does not need sophisticated UI/UX.

The application should contain enough functionality to demonstrate:

* Frontend → API communication
* API → database communication
* Authentication or another meaningful user workflow
* Successful and failed API requests
* Database-backed operations

---

# 3. Backend Requirements

## 3.1 Django

The backend must be implemented using:

* Python
* Django
* Django REST Framework

Requirements:

* Provide REST APIs consumed by the Next.js frontend.
* Implement database-backed functionality.
* Provide application health checks.
* Handle application configuration through environment variables or managed configuration.
* Do not store credentials in source code.
* Produce structured/application logs.
* Return appropriate HTTP status codes.
* Support production deployment.
* Be independently deployable from the frontend.

A health endpoint such as:

```text
/health
```

should be available for infrastructure and deployment validation.

---

# 4. Database Requirements

## 4.1 PostgreSQL

The application must use **PostgreSQL**.

Production PostgreSQL must be hosted using **Amazon RDS for PostgreSQL**.

Requirements:

* Database must not be publicly accessible.
* Database must reside in private subnets.
* Application servers must access the database through restricted security-group rules.
* Database credentials must not be committed to Git.
* Database credentials must be managed through AWS-managed secrets/configuration.
* Database storage must support backups.
* Database health must be monitored.
* Database capacity and connection usage must be observable.
* Database restoration must be tested.

---

# 5. AWS Infrastructure Requirements

The application must be deployed to AWS.

The infrastructure must be reproducible from code.

The target architecture should contain:

```text
                         Internet
                            |
                            v
                       CloudFront
                            |
                            v
                    Application Load
                       Balancer
                            |
              +-------------+-------------+
              |                           |
              v                           v
        Next.js Web Tier             Django API Tier
        EC2 Auto Scaling             EC2 Auto Scaling
              |                           |
              +-------------+-------------+
                            |
                            v
                    PostgreSQL RDS
```

The architecture must be designed for multiple Availability Zones.

---

# 6. Networking Requirements

Create a dedicated AWS VPC.

The network must contain:

* Multiple Availability Zones
* Public subnets
* Private application subnets
* Private database subnets
* Internet Gateway where required
* NAT Gateway or equivalent outbound access mechanism where required
* Route tables
* Security Groups
* Network isolation between tiers

### Network rules

The architecture must follow these principles:

```text
Internet
   |
   v
Load Balancer
   |
   +----> Next.js
   |
   +----> Django
             |
             v
          PostgreSQL
```

Requirements:

* PostgreSQL must not be directly accessible from the Internet.
* Application servers must not unnecessarily expose public ports.
* Only required ports should be allowed between tiers.
* Database access should only be permitted from the appropriate application security group.
* Administrative access must be restricted.
* Network access should follow least-privilege principles.

---

# 7. High Availability Requirements

The application must continue operating when an individual application server fails.

Requirements:

* Deploy application instances across multiple Availability Zones.
* Use multiple instances for the frontend and backend tiers.
* Place instances behind a load balancer.
* Configure health checks.
* Automatically remove unhealthy instances from service.
* Automatically replace failed instances.
* Ensure traffic can continue through remaining healthy instances.

The infrastructure must not depend on a single EC2 instance.

---

# 8. Load Balancing Requirements

Use an AWS Application Load Balancer.

Requirements:

* Distribute traffic across healthy application instances.
* Perform health checks.
* Stop routing traffic to unhealthy instances.
* Support HTTPS.
* Provide access logs and metrics.
* Operate across multiple Availability Zones.
* Support the application's deployment strategy.

The load balancer must not route production traffic to an unhealthy application instance.

---

# 9. Auto Scaling Requirements

Application tiers must support horizontal scaling.

Requirements:

* Define minimum instance count.
* Define desired instance count.
* Define maximum instance count.
* Automatically replace unhealthy instances.
* Support scaling based on appropriate infrastructure/application metrics.
* Ensure new instances receive the correct application version and configuration automatically.
* No manual server configuration should be required when an instance is created.

The system should be capable of handling increased traffic by adding application capacity.

---

# 10. Infrastructure as Code Requirements

All AWS infrastructure must be managed using **Terraform**.

Terraform must manage, where applicable:

* VPC
* Subnets
* Route tables
* Internet/NAT gateways
* Security groups
* IAM roles/policies
* EC2 instances or launch templates
* Auto Scaling Groups
* Application Load Balancer
* Target groups
* RDS
* CloudFront
* CloudWatch resources
* Secrets/configuration resources
* Backup-related resources

Requirements:

* No manual creation of production infrastructure.
* Infrastructure changes must be reviewable through Git.
* Infrastructure must be reproducible.
* Terraform state must not be committed to Git.
* Terraform state should use appropriate remote storage and locking.
* Reusable Terraform modules should be used where appropriate.
* Environment-specific configuration must be separated from reusable infrastructure definitions.

---

# 11. Environment Requirements

The project should support separate environments:

```text
development
staging
production
```

Requirements:

* Environments must be isolated.
* Infrastructure should be reproducible.
* Environment-specific configuration must not be hard-coded.
* Production credentials must never be reused in development.
* Deployment processes should be consistent between environments.
* Changes should be validated in staging before production deployment.

---

# 12. Configuration and Secrets Requirements

Application configuration must be separated from application code.

Requirements:

* No passwords in Git.
* No API keys in Git.
* No database credentials in Git.
* No production secrets inside Docker images or application artifacts.
* Secrets must be retrieved through AWS-managed mechanisms.
* IAM permissions must control access to secrets.
* Different environments must use separate configuration/secrets.

Secrets must never appear in:

* Source code
* Git history
* CI logs
* Application logs
* Terraform state where avoidable

---

# 13. HTTPS Requirements

Production traffic must use HTTPS.

Requirements:

* TLS certificates must be managed using AWS certificate-management services.
* HTTPS must be supported at the appropriate edge/load-balancing layer.
* HTTP traffic should redirect to HTTPS.
* Certificate renewal should be automated.
* Internal communication should use secure protocols where appropriate.

---

# 14. CDN Requirements

Use **Amazon CloudFront**.

Requirements:

* Serve appropriate frontend/static content through the CDN.
* Support HTTPS.
* Configure an appropriate origin.
* Configure caching behavior.
* Prevent incorrect caching of dynamic/API responses.
* Monitor CDN traffic and errors.
* Document the CloudFront architecture and caching strategy.

---

# 15. CI Requirements

Use **GitHub Actions** for Continuous Integration.

Every relevant code change should trigger automated validation.

The CI pipeline should perform appropriate checks such as:

### Frontend

* Install dependencies
* Lint
* Unit tests
* Production build
* Dependency/security checks

### Backend

* Install Python dependencies
* Lint
* Unit tests
* Django checks
* API/integration tests
* Dependency/security checks

### Infrastructure

* Terraform formatting
* Terraform validation
* Terraform plan
* Infrastructure/security checks

CI must fail when required checks fail.

---

# 16. Automated Testing Requirements

The project must include automated tests.

Required test categories:

### Unit Tests

Test important application functionality independently.

### API Tests

Test Django REST Framework endpoints.

### Integration Tests

Validate interaction between application components where appropriate.

### Health Checks

Verify that deployed services are operational.

### Smoke Tests

After deployment, automatically verify critical functionality.

The deployment pipeline must not consider a deployment successful merely because the server started.

---

# 17. Continuous Deployment Requirements

GitHub Actions must support automated deployments.

Target flow:

```text
Git Push
   |
   v
CI
   |
   +--> Tests
   +--> Security Checks
   +--> Build
   |
   v
Staging Deployment
   |
   v
Smoke Tests
   |
   v
Production Deployment
```

Requirements:

* Production deployments must be automated.
* Deployment should not depend on manually SSHing into servers.
* Application versions must be identifiable.
* Deployment failures must be detectable.
* Deployment logs must be available.
* Production deployment should require appropriate approval controls where applicable.

---

# 18. Zero-Downtime Deployment Requirements

Production deployments must avoid unnecessary downtime.

The deployment strategy should support one or more of:

* Rolling deployment
* Blue/green deployment
* Canary deployment

Requirements:

* Existing healthy instances continue serving traffic while new versions are deployed.
* New instances must pass health checks before receiving traffic.
* Unhealthy deployments must not receive production traffic.
* Deployment progress must be observable.
* Failed deployments must support rollback.
* Database migrations must be designed with deployment compatibility in mind.

---

# 19. Rollback Requirements

The system must support application rollback.

Requirements:

* Previous application versions must be identifiable.
* A failed deployment must be detectable.
* The previous known-good version must be recoverable.
* Rollback steps must be documented.
* Rollback should be tested rather than existing only as documentation.

---

# 20. Logging Requirements

Application and infrastructure logs must be centralized.

Logs should include, where appropriate:

* Next.js/frontend logs
* Django application logs
* API request logs
* Load balancer logs
* EC2/system logs
* Deployment logs
* Infrastructure-related logs
* Security/audit logs

Requirements:

* Logs must not exist only on individual EC2 machines.
* Logs must be searchable.
* Logs must contain timestamps.
* Log retention must be defined.
* Sensitive information must not be logged.
* Logs must be useful for troubleshooting production incidents.

The system should allow an operator to answer:

> What happened during an incident?

---

# 21. Monitoring Requirements

Use AWS monitoring capabilities to monitor the infrastructure and application.

Monitor appropriate metrics for:

### EC2

* CPU utilization
* Memory utilization
* Disk usage
* Network activity
* Instance health

### Load Balancer

* Request count
* Request latency
* HTTP 4xx errors
* HTTP 5xx errors
* Healthy targets
* Unhealthy targets

### Django/API

* Request rate
* Error rate
* Response latency
* Health status

### PostgreSQL/RDS

* CPU utilization
* Database connections
* Storage usage
* Read/write activity
* Database latency
* Database health

### CloudFront

* Requests
* Error rates
* Cache behavior
* Traffic volume

---

# 22. Historical Metrics Requirements

Monitoring must retain enough historical data to investigate incidents.

The system should make it possible to determine:

* When an incident started
* Traffic volume before and during the incident
* CPU behavior
* Memory behavior
* Database behavior
* API latency
* HTTP error rates
* Instance scaling activity
* Deployment activity
* Recovery time

An operator should be able to correlate:

```text
Deployment
    |
    v
Increased Errors
    |
    v
Increased Latency
    |
    v
Rollback
    |
    v
Recovery
```

---

# 23. Alerting Requirements

Configure alerts for important operational conditions.

Examples include:

* High CPU utilization
* High memory utilization
* High disk usage
* High API latency
* High HTTP 5xx rate
* Large increase in HTTP 4xx errors
* Unhealthy application instances
* Load balancer target failures
* Database storage problems
* High database CPU
* Excessive database connections
* Backup failures
* Application health-check failures

Alerts should provide enough information for an operator to begin investigating the problem.

---

# 24. Backup Requirements

Production database data must be backed up automatically.

Requirements:

* Automated database backups.
* At least daily backup capability.
* Defined backup retention.
* Point-in-time recovery where supported.
* Backup monitoring.
* Backup failure visibility.
* Documented restore procedure.

Backups must not be considered complete until restoration has been tested.

---

# 25. Restore Testing Requirements

Perform an actual database restoration test.

The project should demonstrate:

```text
Production Database
        |
        v
      Backup
        |
        v
   Restore Database
        |
        v
Application Connection
        |
        v
Data Verification
```

Document:

* Restoration procedure
* Restoration duration
* Data verification procedure
* Potential data loss
* Dependencies
* Recovery limitations

---

# 26. Disaster Recovery Requirements

Document the disaster recovery strategy.

The documentation must define:

### RPO

How much data loss is acceptable after a major failure.

### RTO

How long the system can take to recover.

The project should document:

* Backup strategy
* Restore strategy
* Infrastructure recreation
* Database recovery
* Application deployment
* DNS/CDN considerations
* Recovery dependencies
* Recovery limitations

Where practical, recovery procedures should be tested.

---

# 27. Security Requirements

The infrastructure must follow basic production security principles.

Requirements:

* Least-privilege IAM.
* Restricted security groups.
* Private database.
* HTTPS.
* Managed secrets.
* No credentials in Git.
* No unnecessary public ports.
* Restricted administrative access.
* Regular dependency updates.
* Infrastructure security validation.
* Audit logging where appropriate.
* Appropriate OS patching.
* Secure CI/CD credentials.

---

# 28. IAM Requirements

AWS permissions must follow least privilege.

Separate permissions should be considered for:

* Infrastructure provisioning
* CI/CD
* Application runtime
* Monitoring
* Database access
* Administrative operations

Prefer:

* IAM roles
* Short-lived credentials
* OIDC-based GitHub Actions authentication where appropriate

Avoid long-lived AWS credentials stored unnecessarily in GitHub secrets.

---

# 29. Security Scanning Requirements

Automated security checks should be included in CI.

The project should scan:

* Python dependencies
* Node.js dependencies
* Application source where appropriate
* Terraform configuration
* Container configuration when containers are introduced
* Infrastructure configuration

The pipeline should identify known vulnerabilities and insecure infrastructure configurations before production deployment.

---

# 30. Failure Testing Requirements

The project must intentionally test failure scenarios.

At minimum, test scenarios such as:

### Application Server Failure

Terminate an application instance.

Expected result:

* Load balancer stops sending traffic to it.
* Remaining instances continue serving traffic.
* Auto Scaling replaces the failed instance.

### Application Failure

Make an application process unhealthy.

Expected result:

* Health checks detect the failure.
* Traffic is removed from the unhealthy target.
* Service remains available through healthy targets.

### Traffic Spike

Generate increased traffic.

Expected result:

* Application remains operational.
* Monitoring detects increased load.
* Auto Scaling can increase capacity.

### Deployment Failure

Deploy a deliberately broken application version.

Expected result:

* Deployment failure is detected.
* Broken version does not remain serving production traffic.
* Rollback or recovery procedure restores the previous version.

### Database Recovery

Restore the database from a backup.

Expected result:

* Database can be restored.
* Application can reconnect.
* Data can be verified.

Each failure test must be documented with:

* Scenario
* Expected behavior
* Actual behavior
* Evidence
* Recovery procedure
* Lessons learned

---

# 31. Observability Requirements

The project should provide enough visibility to answer:

### What is broken?

Through:

* Alerts
* Health checks
* Error rates
* Logs

### Where is it broken?

Through:

* Load balancer metrics
* Application metrics
* Database metrics
* Infrastructure metrics

### When did it break?

Through:

* Historical metrics
* Centralized logs
* Deployment history

### What changed?

Through:

* Git history
* Deployment records
* Infrastructure changes
* Configuration changes

### Is it recovering?

Through:

* Current metrics
* Error rates
* Health checks
* Application logs

---

# 32. Documentation Requirements

The repository must contain documentation covering:

* Project architecture
* AWS infrastructure
* Network architecture
* Security model
* Terraform architecture
* Environment setup
* Application deployment
* CI/CD pipeline
* Zero-downtime deployment strategy
* Rollback procedure
* Monitoring
* Logging
* Alerting
* Backup strategy
* Database restoration
* Disaster recovery
* Failure testing
* Troubleshooting
* Operational procedures

Documentation should be written so that another engineer can understand and operate the system.

---

# 33. Repository Requirements

Suggested repository structure:

```text
project/
│
├── frontend/
│   └── Next.js application
│
├── backend/
│   └── Django / Django REST Framework application
│
├── terraform/
│   ├── modules/
│   └── environments/
│       ├── development/
│       ├── staging/
│       └── production/
│
├── tests/
│
├── scripts/
│
├── monitoring/
│
├── docs/
│
├── .github/
│   └── workflows/
│
├── README.md
│
└── .gitignore
```

The exact structure may change as the project evolves.

---

# 34. Operational Requirements

The final system should allow an engineer to perform common operational tasks without manually configuring individual servers.

Required operational capabilities include:

* Deploy application
* Roll back application
* Scale application
* Replace failed instance
* View application logs
* View infrastructure metrics
* Investigate alerts
* Restore database
* Verify backups
* Recreate infrastructure
* Validate application health
* Perform failure tests

---

# 35. Definition of Done

The project is considered complete when:

## Application

* [ ] Next.js frontend is functional.
* [ ] Django REST API is functional.
* [ ] PostgreSQL database is integrated.
* [ ] Frontend communicates with backend.
* [ ] Backend communicates with database.
* [ ] Health endpoints exist.

## AWS

* [ ] AWS VPC created.
* [ ] Multiple Availability Zones used.
* [ ] Public/private subnet separation implemented.
* [ ] Application Load Balancer configured.
* [ ] EC2 Auto Scaling configured.
* [ ] RDS PostgreSQL configured.
* [ ] CloudFront configured.
* [ ] HTTPS configured.

## Terraform

* [ ] Infrastructure is defined in Terraform.
* [ ] Terraform state is managed safely.
* [ ] Reusable modules are used where appropriate.
* [ ] Environments are separated.
* [ ] Infrastructure can be recreated from code.

## CI/CD

* [ ] GitHub Actions configured.
* [ ] Automated tests run.
* [ ] Security checks run.
* [ ] Infrastructure validation runs.
* [ ] Application builds automatically.
* [ ] Staging deployment is automated.
* [ ] Production deployment is automated.
* [ ] Smoke tests run after deployment.
* [ ] Rollback procedure exists.

## Availability

* [ ] Multiple application instances run.
* [ ] Instances span multiple Availability Zones.
* [ ] Load balancer health checks work.
* [ ] Failed instances are automatically replaced.
* [ ] Traffic continues during individual instance failure.
* [ ] Auto Scaling has been tested.

## Observability

* [ ] Application logs are centralized.
* [ ] Infrastructure logs are available.
* [ ] Metrics are collected.
* [ ] Historical metrics are available.
* [ ] Alerts are configured.
* [ ] Incident investigation is possible using logs and metrics.

## Backup & Recovery

* [ ] Automated database backups configured.
* [ ] Backup retention defined.
* [ ] Restore procedure documented.
* [ ] Database restoration tested.
* [ ] Disaster recovery procedure documented.
* [ ] RPO defined.
* [ ] RTO defined.

## Security

* [ ] Database is private.
* [ ] Security groups follow least privilege.
* [ ] HTTPS is enforced.
* [ ] Secrets are managed securely.
* [ ] No credentials exist in Git.
* [ ] IAM permissions follow least privilege.
* [ ] Dependency security checks exist.
* [ ] Infrastructure security checks exist.

## Failure Testing

* [ ] EC2 failure tested.
* [ ] Application failure tested.
* [ ] Traffic spike tested.
* [ ] Deployment failure tested.
* [ ] Rollback tested.
* [ ] Database restore tested.
* [ ] Results documented.

---

# 36. Final Success Criteria

The final system should demonstrate that the application can:

```text
BUILD
  ↓
TEST
  ↓
PROVISION
  ↓
DEPLOY
  ↓
SCALE
  ↓
MONITOR
  ↓
DETECT FAILURE
  ↓
RECOVER
  ↓
BACK UP
  ↓
RESTORE
  ↓
ROLL BACK
```

The primary goal is not to build a complex application.

The primary goal is to demonstrate the ability to **operate a real application reliably in AWS using automation, Infrastructure as Code, CI/CD, monitoring, security, backups, scaling, and failure recovery**.