# Ansible Server Provisioning & Automation

A practical Ansible automation project demonstrating inventory-driven Linux server provisioning, idempotent configuration management, reusable roles, and AWS API-based EC2 provisioning.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/52bfd44a-d015-4a01-82c1-90a333981ae7" />


## Overview

This project demonstrates how Ansible can be used to move from one-off infrastructure operations to structured and reusable automation.

The implementation covers:

- SSH-based Linux host management
- YAML inventory and hierarchical host grouping
- Ad hoc Ansible operations
- Repeatable playbooks
- Idempotent package, service, and file management
- Variables and runtime facts
- OS-specific conditional execution
- Loops for repeated operations
- Jinja2 configuration templates
- Change-triggered handlers
- Reusable Ansible roles
- AWS API-based EC2 automation

The project focuses on practical automation and configuration management rather than application development.

## Ownership & Scope

I personally implemented and validated the Ansible automation represented in this repository, including inventory configuration, playbook development, variable and fact handling, conditional execution, loops, configuration templating, handlers, role organization, troubleshooting, and AWS API-based provisioning.

The repository intentionally does not reproduce course material or third-party/community role source code.

Community roles and collections used during the practical are treated as external dependencies rather than original work.

## Architecture

At a high level:

```text
                    Ansible Control Machine
                             │
                 ┌───────────┴───────────┐
                 │                       │
             SSH / Modules          AWS APIs
                 │                       │
                 ▼                       ▼
          Linux Managed Hosts       AWS Resources
                 │                       │
        ┌────────┼────────┐              │
        │        │        │              │
      Web      Web       DB             EC2
      Host     Host      Host          Resources
```

The automation is organized around:

```text
Inventory
    ↓
Target Selection
    ↓
Playbooks
    ↓
Modules
    ↓
Desired State
    ↓
Variables / Facts
    ↓
Conditions / Loops
    ↓
Templates
    ↓
Handlers
    ↓
Reusable Roles
```

See [Architecture](docs/architecture.md) for the detailed execution model.

## Engineering Contribution

The strongest engineering work demonstrated by this project includes:

### Inventory & Targeting

Configured hosts and hierarchical groups so automation can target individual machines, functional groups, or larger infrastructure groups.

### Idempotent Configuration

Used Ansible's desired-state model for package, service, and file management so repeated execution does not unnecessarily modify systems.

### Parameterized Automation

Separated automation logic from changing data using variables, group variables, host variables, and runtime facts.

### OS-Aware Provisioning

Used runtime facts and conditional execution to apply different package, service, and configuration operations to different operating systems.

### Dynamic Configuration

Used Jinja2 templates to generate configuration files from variables rather than maintaining separate hardcoded configurations.

### Event-Driven Actions

Used handlers so dependent actions such as service restarts occur when the relevant configuration actually changes.

### Reusable Roles

Converted working provisioning logic into an Ansible role structure containing tasks, handlers, defaults, templates, and files.

### AWS Automation

Used Ansible AWS modules to interact with AWS APIs, create an EC2 key pair, capture newly generated key material, and launch EC2 instances using an idempotent instance-count model.

See [Implementation](docs/implementation.md) for the detailed implementation.

## Validation

Validation focused on:

- SSH connectivity
- Host and group targeting
- Successful playbook execution
- Idempotent repeated execution
- OS-specific task selection
- Variable and fact resolution
- Template change detection
- Handler execution after configuration changes
- Role execution
- AWS resource creation and idempotent instance management

See [Validation](docs/validation.md) for the validation strategy and evidence mapping.

## Project Boundaries

This project demonstrates practical Ansible automation but does not establish:

- Production-grade Ansible controller architecture
- Enterprise secrets management
- CI/CD implementation
- Terraform-based infrastructure provisioning
- Kubernetes automation
- Production monitoring
- Disaster recovery
- Enterprise AWS governance
- Production-scale reliability

AWS credentials, private keys, and other sensitive environment-specific values are intentionally excluded from the repository.

See [Limitations & Future Work](docs/limitations-and-future-work.md).

## Technologies

- Ansible
- YAML
- Jinja2
- Linux
- SSH
- AWS EC2
- AWS IAM
- boto3
- Ansible Collections
- Ansible Galaxy

## Repository Navigation

- [Architecture](docs/architecture.md)
- [Implementation](docs/implementation.md)
- [Validation](docs/validation.md)
- [Limitations & Future Work](docs/limitations-and-future-work.md)

## Evidence

High-signal evidence from the completed environment is maintained separately under:

```text
evidence/screenshots/
```

Only personal execution evidence should be presented as proof of project execution.

Course screenshots, lecture material, credentials, and third-party source code are not included as project evidence.

## Future Direction

Potential extensions include:

```text
Current Ansible Automation
        ↓
Role Testing
        ↓
CI Validation
        ↓
Secrets Management
        ↓
Ansible Automation Platform / AWX
        ↓
Terraform Infrastructure Provisioning
        ↓
Ansible Configuration Management
```

These are future improvements and are not capabilities claimed by the current implementation.
