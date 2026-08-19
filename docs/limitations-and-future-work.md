# Limitations & Future Work

[← Back to README](../README.md) | [Architecture](architecture.md) | [Implementation](implementation.md) | [Validation](validation.md)

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/dd8ee280-eac2-4ea0-931f-6fa2b02d9e9f" />

---

## 1. Overview

This project demonstrates practical Ansible automation for Linux server provisioning, configuration management, reusable roles, and AWS API-based resource provisioning.

The implementation intentionally focuses on the core Ansible concepts demonstrated in the learning material.

It should therefore be viewed as a **practical learning and portfolio project**, not as a complete enterprise automation platform.

The main limitations are:

```text
Current Project
      │
      ├── Limited production hardening
      ├── Limited secrets management
      ├── Limited automated testing
      ├── Limited CI/CD integration
      ├── Limited centralized control
      ├── Limited AWS governance
      └── Limited production-scale operations
```

Future work can progressively address these limitations without changing the core architecture.

---

# 2. Current Scope

The current implementation demonstrates:

- Ansible control-machine architecture.
- SSH-based Linux host management.
- Inventory-driven targeting.
- Host and group variables.
- Ad hoc Ansible modules.
- YAML playbooks.
- Idempotent configuration management.
- Runtime facts.
- Conditional execution.
- Loops.
- Jinja2 templates.
- Handlers.
- Reusable Ansible roles.
- Ansible Galaxy/community role usage for learning.
- AWS key-pair automation.
- AWS EC2 provisioning.
- Basic troubleshooting and validation.

The project does not attempt to implement every capability available in the Ansible ecosystem.

---

# 3. Limitation — No Production Ansible Controller

The current project uses a local Ansible control machine.

Conceptually:

```text
Developer / Operator
        │
        ▼
Ansible Control Machine
        │
        ▼
Managed Hosts
```

There is no production-grade centralized Ansible Automation Controller or AWX deployment established by this project.

### Impact

There is no demonstrated:

- Central job dashboard.
- Centralized credential management.
- Role-based access control.
- Central job history.
- Scheduled automation platform.
- Enterprise workflow management.

### Future Work

A future iteration could introduce:

```text
Users
  ↓
AWX / Ansible Automation Platform
  ↓
Job Templates
  ↓
Credentials
  ↓
Inventories
  ↓
Managed Infrastructure
```

This would provide centralized execution and operational visibility.

---

# 4. Limitation — Secrets Management

The current project intentionally keeps:

- AWS credentials
- SSH private keys
- Passwords
- Tokens

outside the repository.

However, a complete enterprise secrets-management architecture has not been implemented.

The current boundary is:

```text
Repository
    ≠
Secrets
```

### Future Work

Possible future improvements include:

```text
Ansible Vault
      ↓
Encrypted Variables
```

and eventually integration with a dedicated secrets-management platform.

The objective would be:

```text
Automation
    ↓
Secure Credential Retrieval
    ↓
Target / Cloud API
```

without exposing credentials in source control.

---

# 5. Limitation — No CI/CD Integration

The current project does not establish a complete CI/CD pipeline for Ansible validation.

The current workflow is primarily:

```text
Developer
   ↓
Edit Playbook
   ↓
Run Ansible
   ↓
Validate
```

### Future Work

A CI pipeline could automatically perform:

```text
Git Push
   ↓
Lint
   ↓
Syntax Check
   ↓
Ansible Validation
   ↓
Automated Tests
   ↓
Approval
   ↓
Deployment
```

Potential validation stages could include:

- YAML validation.
- Ansible syntax checks.
- Ansible linting.
- Role testing.
- Molecule-based testing.
- Security checks.

This would move the project from manually validated automation toward continuously validated automation.

---

# 6. Limitation — Limited Automated Testing

The current validation relies heavily on executing playbooks against actual infrastructure and inspecting the results.

This is useful, but it is not equivalent to a comprehensive automated test suite.

The current model is:

```text
Playbook
   ↓
Real Environment
   ↓
Observe Result
```

### Future Work

A future implementation could introduce automated role testing:

```text
Role
 ↓
Test Environment
 ↓
Converge
 ↓
Verify
 ↓
Destroy
```

Molecule could be evaluated for this purpose.

This would allow reusable roles to be tested consistently before being applied to real infrastructure.

---

# 7. Limitation — Limited Production Hardening

The project focuses on learning and demonstrating Ansible concepts.

It does not establish comprehensive production hardening for:

- SSH.
- Linux operating systems.
- AWS accounts.
- IAM policies.
- Network security.
- Service configuration.
- Secrets.
- Logging.
- Monitoring.

### Future Work

A production-oriented extension could introduce a dedicated hardening role:

```text
base-hardening/
├── tasks/
├── handlers/
├── defaults/
├── templates/
└── files/
```

Possible responsibilities could include:

```text
SSH hardening
Firewall configuration
Package updates
User management
Sudo policy
Audit configuration
Security baseline
```

The exact hardening controls would need to be defined according to the target environment rather than assumed universally.

---

# 8. Limitation — Limited AWS Infrastructure Scope

The AWS portion demonstrates API-based EC2 provisioning.

It does not establish a complete AWS infrastructure platform.

The demonstrated scope is approximately:

```text
Ansible
   ↓
AWS API
   ├── EC2 Key Pair
   └── EC2 Instance
```

It does not establish comprehensive management of:

- VPCs.
- Subnets.
- Route tables.
- NAT gateways.
- Load balancers.
- Auto Scaling Groups.
- RDS.
- IAM architecture.
- Multi-account governance.

### Future Work

Additional AWS automation could be introduced incrementally.

However, the learning material positions Terraform as the more specialized tool for heavy cloud infrastructure management.

Therefore a future architecture could separate responsibilities:

```text
Terraform
    ↓
Cloud Infrastructure
    ↓
VPC / Subnets / Security / EC2
    ↓
Ansible
    ↓
Operating System Configuration
    ↓
Application Configuration
```

This would provide clearer infrastructure-versus-configuration boundaries.

---

# 9. Limitation — No Terraform Integration

The current project uses Ansible for the demonstrated AWS provisioning tasks.

Terraform has not been integrated into this repository.

This is intentionally outside the current implementation scope.

### Future Work

A future DevOps iteration could introduce:

```text
Terraform
    ↓
Provision AWS infrastructure
    ↓
Outputs
    ↓
Ansible Inventory
    ↓
Configure Servers
```

The resulting workflow would be:

```text
Infrastructure as Code
        ↓
Terraform
        ↓
Cloud Resources
        ↓
Configuration Management
        ↓
Ansible
```

This would more clearly demonstrate the complementary responsibilities of Terraform and Ansible.

---

# 10. Limitation — Static Inventory

The current project uses a repository-managed inventory.

This is appropriate for the demonstrated environment, but infrastructure that changes frequently can make static inventory harder to maintain.

Current model:

```text
inventory.yml
      ↓
Static Host Definitions
```

### Future Work

A dynamic infrastructure model could use:

```text
AWS
 ↓
Dynamic Inventory
 ↓
Ansible
```

This would allow Ansible to discover EC2 instances based on attributes such as:

- Tags.
- Regions.
- Environment.
- Roles.

This becomes increasingly useful as infrastructure grows.

---

# 11. Limitation — Limited Environment Separation

The current repository does not establish a complete environment hierarchy such as:

```text
development
staging
production
```

### Future Work

A more mature repository could use:

```text
inventories/
├── dev/
├── staging/
└── production/
```

with environment-specific variables and inventories.

The resulting structure could be:

```text
Common Role
     │
     ├── Development values
     ├── Staging values
     └── Production values
```

This would allow the same automation logic to be reused across environments while keeping environment-specific data separate.

---

# 12. Limitation — Limited Role Library

The project contains the demonstrated `post-install` role.

It does not attempt to build a large reusable role ecosystem.

### Future Work

The automation could eventually be separated into focused roles:

```text
roles/
├── base/
├── security/
├── users/
├── packages/
├── ntp/
├── webserver/
├── database/
└── monitoring/
```

This would make individual responsibilities easier to reuse and test.

The important constraint would be to avoid creating roles simply for the sake of creating more roles.

Roles should represent meaningful reusable boundaries.

---

# 13. Limitation — Limited Error Handling

The current project primarily relies on Ansible's normal task failure behavior.

A production automation platform may need more sophisticated failure handling.

### Future Work

Possible improvements include:

```text
Task Failure
     ↓
rescue
     ↓
Recovery / Cleanup
     ↓
always
     ↓
Final State / Notification
```

Additional patterns could include:

- Explicit validation tasks.
- Preconditions.
- Postconditions.
- Controlled retries.
- Timeouts.
- Rollback workflows.
- Failure notifications.

These should be introduced where they solve a real operational requirement rather than adding unnecessary complexity.

---

# 14. Limitation — Limited Observability

The current project validates execution through:

- Ansible output.
- Play recaps.
- Target state.
- AWS resource inspection.
- Screenshots.

It does not establish centralized monitoring or observability.

### Future Work

A production-oriented architecture could integrate:

```text
Ansible
   ↓
Execution Results
   ↓
Central Logging
   ↓
Monitoring / Alerting
```

Potential observability areas include:

- Automation failures.
- Configuration drift.
- Server health.
- Service state.
- Deployment status.

---

# 15. Limitation — Manual Evidence Collection

The project uses an evidence directory:

```text
evidence/
└── screenshots/
```

Evidence is manually collected from relevant executions.

### Future Work

A more mature workflow could automatically capture:

```text
CI/CD Run
    ↓
Test Results
    ↓
Artifacts
    ↓
Validation Report
```

This would make evidence reproducible and reduce reliance on manually captured screenshots.

---

# 16. Limitation — No Formal Configuration Drift System

Ansible can enforce desired state when it is executed, but this project does not establish a continuous configuration-drift detection system.

Current model:

```text
Run Ansible
     ↓
Correct Configuration
```

### Future Work

A future system could periodically evaluate:

```text
Desired State
      ↓
Actual State
      ↓
Drift?
   /     No      Yes
         ↓
      Alert / Remediate
```

This could be integrated with a centralized automation controller and monitoring platform.

---

# 17. Limitation — No Production Deployment Strategy

The project demonstrates provisioning and configuration but does not establish a complete production deployment strategy.

There is no demonstrated:

- Blue/green deployment.
- Canary deployment.
- Rolling application deployment.
- Automated rollback.
- Deployment approval workflow.

### Future Work

Application deployment could eventually follow:

```text
Build
  ↓
Artifact
  ↓
Test
  ↓
Approval
  ↓
Ansible Deployment
  ↓
Health Check
  ↓
Success / Rollback
```

This would extend the project from server configuration into application delivery.

---

# 18. Limitation — No Kubernetes Automation

Kubernetes is outside the scope of the current project.

The current architecture focuses on:

```text
Linux Servers
      +
AWS EC2
```

### Future Work

Kubernetes could be introduced as a separate automation layer:

```text
Terraform
   ↓
Infrastructure
   ↓
Kubernetes Cluster
   ↓
Ansible
   ↓
Supporting Configuration / Operations
```

The project should not claim Kubernetes automation until such workflows are actually implemented and validated.

---

# 19. Limitation — No Enterprise AWS Governance

The AWS automation uses AWS APIs but does not establish an enterprise governance model.

It does not demonstrate:

- Multi-account strategy.
- Organizational policies.
- Centralized IAM governance.
- Cost controls.
- Resource tagging standards.
- Compliance policies.
- Service Control Policies.

### Future Work

A future cloud platform could introduce:

```text
AWS Organization
      ↓
Accounts
      ↓
IAM / Policies
      ↓
Networking
      ↓
Infrastructure
      ↓
Configuration Management
```

This would be a substantially larger scope than the current project.

---

# 20. Limitation — No Disaster Recovery Design

The project does not establish disaster recovery.

There is no demonstrated:

- Backup architecture.
- Cross-region recovery.
- Recovery Point Objective.
- Recovery Time Objective.
- Automated restoration.

### Future Work

A future project could explicitly define:

```text
Failure
  ↓
Detection
  ↓
Recovery Strategy
  ↓
Restore
  ↓
Validation
```

with measurable recovery objectives.

---

# 21. Future Work Roadmap

Future development can be staged rather than implemented all at once.

```text
Current Project
      │
      ▼
Phase 1 — Harden Automation
      │
      ├── Better variable organization
      ├── Stronger validation
      ├── Role cleanup
      └── Secrets handling
      │
      ▼
Phase 2 — Test Automation
      │
      ├── Ansible Lint
      ├── Molecule
      ├── Automated verification
      └── CI validation
      │
      ▼
Phase 3 — Centralize Execution
      │
      ├── AWX / Ansible Automation Platform
      ├── Central credentials
      ├── Job templates
      └── Scheduling
      │
      ▼
Phase 4 — Infrastructure Separation
      │
      ├── Terraform
      ├── AWS infrastructure
      └── Dynamic inventory
      │
      ▼
Phase 5 — Production Operations
      │
      ├── Monitoring
      ├── Drift detection
      ├── Failure recovery
      └── Deployment automation
```

---

# 22. Recommended Next Iteration

The highest-value next step is not to immediately add more AWS resources or more Ansible tasks.

The better progression is:

```text
Current Role
    ↓
Test the Role
    ↓
Add CI Validation
    ↓
Secure Secrets
    ↓
Centralize Execution
    ↓
Introduce Terraform
```

This builds engineering maturity around the existing automation before expanding its infrastructure scope.

---

# 23. Future Architecture

A mature version of this project could evolve toward:

```text
                    Developer
                       │
                       ▼
                    Git Repo
                       │
                       ▼
                  CI Pipeline
                       │
            ┌──────────┴──────────┐
            │                     │
        Terraform              Ansible
            │                     │
            ▼                     ▼
      AWS Infrastructure    Server Configuration
            │                     │
            └──────────┬──────────┘
                       │
                       ▼
                 Applications
                       │
                       ▼
              Monitoring / Logs
                       │
                       ▼
                  Operations
```

The separation is:

```text
Terraform
    =
Infrastructure

Ansible
    =
Configuration

CI/CD
    =
Validation & Delivery

Monitoring
    =
Operational Visibility
```

This is a future architectural direction, not a capability claimed by the current project.

---

# 24. Engineering Maturity Progression

The project can be viewed as an incremental maturity path:

```text
Level 1
Manual Server Operations
        ↓
Level 2
Ansible Ad Hoc Commands
        ↓
Level 3
Ansible Playbooks
        ↓
Level 4
Variables + Facts + Conditions + Loops
        ↓
Level 5
Templates + Handlers
        ↓
Level 6
Reusable Roles
        ↓
Level 7
Tested Automation
        ↓
Level 8
CI/CD Automation
        ↓
Level 9
Centralized Automation
        ↓
Level 10
Infrastructure + Configuration + Operations Platform
```

The current project primarily demonstrates Levels 2–6, with AWS API automation extending the scope into cloud resource management.

---

# 25. What Should Not Be Added Yet

Future work should remain controlled.

The project should **not** immediately add every possible DevOps technology simply to increase the technology list.

Avoid prematurely adding:

```text
Kubernetes
Terraform
Jenkins
AWX
Vault
Prometheus
Grafana
Argo CD
Service Mesh
```

unless the next iteration has a clear engineering objective that requires them.

A stronger portfolio project demonstrates:

```text
Problem
  ↓
Requirement
  ↓
Design
  ↓
Implementation
  ↓
Validation
  ↓
Measured Improvement
```

rather than:

```text
Technology
   +
Technology
   +
Technology
```

---

# 26. Final Limitations

The most important current limitations are:

1. No centralized Ansible controller.
2. No enterprise secrets-management implementation.
3. No automated CI/CD validation.
4. No comprehensive role-testing framework.
5. Limited production hardening.
6. Limited AWS infrastructure scope.
7. No Terraform integration.
8. Static rather than dynamic inventory.
9. Limited environment separation.
10. Limited observability and drift management.
11. No production deployment strategy.
12. No disaster-recovery implementation.
13. No enterprise AWS governance.
14. No Kubernetes automation.

These limitations are intentional boundaries of the current project rather than failures of the implementation.

---

# 27. Final Future Direction

The project should evolve from:

```text
Ansible Learning Project
```

into:

```text
Tested Ansible Automation
        ↓
CI-Validated Configuration Management
        ↓
Centralized Automation
        ↓
Terraform + Ansible Infrastructure Workflow
        ↓
Production-Oriented DevOps Platform
```

The immediate objective should be **quality and repeatability**, not simply increasing the number of technologies.

The long-term goal is to demonstrate that infrastructure automation can be:

- Repeatable.
- Idempotent.
- Modular.
- Testable.
- Secure.
- Observable.
- Version controlled.
- Reproducible.
- Maintainable.

That represents the natural next stage of the engineering practices demonstrated by this project.
