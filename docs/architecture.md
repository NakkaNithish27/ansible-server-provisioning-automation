# Architecture

[← Back to README](../README.md)

---

## 1. Architecture Overview

This project demonstrates Ansible as an **agentless automation and configuration-management system**.

The architecture has two execution paths:

```text
                         Ansible Control Machine
                                  │
                         ┌────────┴────────┐
                         │                 │
                  System Modules      API Modules
                         │                 │
                       SSH                 │
                         │                 │
                         ▼                 ▼
                 Managed Linux Hosts     AWS APIs
                         │                 │
              ┌──────────┼──────────┐      ▼
              │          │          │   AWS Resources
            web01      web02      db01    │
                                      ┌───┴────┐
                                      │  EC2   │
                                      │Resources│
                                      └────────┘
```

For Linux system automation, Ansible connects to managed hosts through SSH and executes modules remotely.

For AWS API-based automation, Ansible runs the API module on the control machine and communicates with AWS through its APIs. The execution mode is determined by the module and target type.

---

## 2. Control Machine

The **control machine** is the machine where Ansible is installed and where playbooks are executed.

The practical architecture uses:

```text
Control Machine
│
├── Ansible
├── Inventory
├── ansible.cfg
├── Playbooks
├── Variables
├── Templates
└── Roles
```

Managed hosts do not require Ansible to be installed.

The control machine uses the existing SSH infrastructure to manage Linux targets. This is the core agentless design demonstrated in the practical.

---

## 3. Inventory Architecture

The inventory acts as the target and connection definition for the automation.

Conceptually:

```text
Inventory
│
├── Hosts
│   ├── web01
│   ├── web02
│   └── db01
│
├── Groups
│   ├── webservers
│   └── dbservers
│
└── Parent Groups
    └── dc_oregon
```

Inventory information includes values such as:

- Target hostname or IP address
- SSH username
- SSH private-key path
- SSH port when required

The practical recommends keeping the inventory with the project rather than depending on the machine-wide `/etc/ansible/hosts` inventory.

### Targeting model

Ansible can target:

```text
Individual host
      ↓
Group
      ↓
Parent group
      ↓
all hosts
      ↓
Pattern
```

For example:

```text
web01
  ↓
webservers
  ↓
dc_oregon
  ↓
all
```

This allows the same automation mechanism to operate at different infrastructure scopes.

---

## 4. SSH Connectivity Layer

For Linux targets, SSH is the transport layer.

The connection flow is:

```text
Ansible
   │
   │ Inventory provides:
   │ ├── host/IP
   │ ├── username
   │ └── private key
   │
   ▼
SSH
   │
   ▼
Managed Linux Host
```

Ansible uses the same fundamental SSH information that would be required for manual access.

The practical also demonstrates project-level configuration such as SSH host-key behavior and private-key permissions.

---

## 5. Ansible Configuration Layer

Project-level configuration is represented by:

```text
ansible.cfg
```

The configuration hierarchy demonstrated in the material is:

```text
ANSIBLE_CONFIG environment variable
             ↓
./ansible.cfg
             ↓
~/.ansible.cfg
             ↓
/etc/ansible/ansible.cfg
```

The repository-level configuration is preferred because it makes project behavior more portable and consistent across environments.

Conceptually:

```text
Project
   │
   ├── ansible.cfg
   ├── inventory
   └── playbooks
```

The configuration can control behavior such as inventory location, host-key checking, parallel execution, and logging.

---

## 6. Playbook and Module Architecture

The main automation abstraction is:

```text
Playbook
   │
   ├── Play
   │    │
   │    └── Tasks
   │          │
   │          └── Modules
   │
   └── Target hosts
```

A playbook describes **what should happen** and **where it should happen**.

A task invokes an Ansible module to perform an individual operation.

Examples demonstrated throughout the project include:

```text
yum / apt
   → package management

service
   → service management

copy
   → static file deployment

template
   → dynamic configuration deployment

file
   → file/directory management

user
   → user management

ec2_key
   → AWS key-pair management

ec2_instance
   → AWS EC2 management
```

Modules are the atomic units of Ansible automation.

---

## 7. Desired-State Execution

The configuration-management architecture is based on declaring the desired state rather than writing a sequence of imperative commands.

Conceptually:

```text
Desired State
     │
     ▼
Ansible Module
     │
     ▼
Current Target State
     │
     ▼
Difference?
   /      No       Yes
 │         │
ok       changed
```

For example, a package task declares:

```text
package must be present
```

Ansible determines whether the target already satisfies that state.

This produces the important execution states:

```text
ok
changed
failed
```

Repeated execution should therefore converge toward the desired state without unnecessarily changing resources.

---

## 8. Variables and Facts

Variables provide the data layer for automation.

The architecture separates:

```text
Automation Logic
       │
       │ references
       ▼
Variables
```

Variables can be defined at different scopes, including:

```text
Playbook variables
       ↓
group_vars
       ↓
host_vars
       ↓
other Ansible variable sources
```

The practical emphasizes separating changing values from task definitions so the automation becomes reusable.

### Facts

Facts provide information automatically gathered from managed hosts.

Conceptually:

```text
Managed Host
     │
     ▼
Gather Facts
     │
     ▼
Host Information
     │
     ├── OS
     ├── CPU
     ├── kernel
     ├── network information
     └── architecture
```

These facts can then influence task execution and configuration.

---

## 9. Conditional Execution

Conditions allow the same playbook to behave differently depending on the target.

The practical uses `when` conditions for OS-specific provisioning:

```text
                    Provisioning Task
                           │
                    ┌──────┴──────┐
                    │             │
                 CentOS         Ubuntu
                    │             │
                  yum           apt
                    │             │
                 chrony          ntp
```

The decision is based on host facts such as:

```text
ansible_distribution
```

This allows a single automation flow to handle different operating-system requirements without duplicating the entire playbook.

---

## 10. Loop Architecture

Loops separate the task definition from the collection of items being processed.

```text
             Task Definition
                    │
                    ▼
                 loop:
              ┌─────┼─────┐
              │     │     │
            item1  item2  item3
              │     │     │
              ▼     ▼     ▼
           Execution per item
```

For example:

```text
loop:
  - chrony
  - wget
  - git
  - zip
  - unzip
```

The same task executes once for each item through:

```text
{{ item }}
```

This removes task duplication and allows the data collection to change independently from the task definition.

---

## 11. Configuration Template Architecture

The project uses Jinja2 templates for configuration that contains dynamic values.

The flow is:

```text
Variables
    │
    ▼
Jinja2 Template
    │
    │ render
    ▼
Final Configuration
    │
    ▼
Managed Host
```

For example:

```text
group_vars/all
      │
      ├── ntp0
      ├── ntp1
      ├── ntp2
      └── ntp3
             │
             ▼
      NTP configuration template
             │
             ▼
      /etc/chrony.conf
      /etc/ntp.conf
```

This creates a separation between:

```text
Configuration structure
        ≠
Configuration values
```

Changing a centralized variable can therefore affect the generated configuration without rewriting the template itself.

---

## 12. Handler Architecture

Handlers introduce a change-triggered execution path.

Without handlers:

```text
Deploy configuration
        ↓
Restart service
        ↓
Every playbook run
```

With handlers:

```text
Deploy configuration
        │
        ▼
Did configuration change?
       /      No   Yes
     │     │
     │     └── notify
     │           ↓
     │       Handler
     │           ↓
     │       Restart service
     │
     └── no restart
```

The handler therefore connects a **change-producing task** to a **dependent action**.

The practical demonstrates the template + handler pattern for restarting services only when the relevant configuration changes.

---

## 13. Role Architecture

Roles provide the project's main reusable automation boundary.

The role structure is:

```text
roles/
└── post-install/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── defaults/
    │   └── main.yml
    ├── vars/
    │   └── main.yml
    ├── templates/
    └── files/
```

The architectural relationship is:

```text
Playbook
    │
    ▼
Role
    │
    ├── Tasks
    ├── Handlers
    ├── Variables
    ├── Defaults
    ├── Templates
    └── Files
```

The role's `tasks/main.yml` acts as the primary entry point.

Ansible automatically resolves role-local templates and files through the standardized directory structure.

### Role benefit

The playbook becomes simpler:

```text
Playbook
   │
   └── roles:
          └── post-install
```

while the implementation details remain organized inside the role.

The practical explicitly converts a multi-component playbook containing tasks, handlers, variables, templates and files into the `post-install` role structure.

---

## 14. AWS API Architecture

The AWS portion uses a different execution path from Linux configuration management.

Instead of:

```text
Control Machine
      │
      │ SSH
      ▼
Linux Host
```

the architecture becomes:

```text
Control Machine
      │
      │ AWS API
      ▼
     AWS
      │
      ├── EC2 Key Pair
      │
      └── EC2 Instance
```

The playbook targets:

```yaml
hosts: localhost
gather_facts: false
```

because the target is AWS rather than a remote operating system.

---

## 15. AWS Key-Pair Flow

The demonstrated key-pair workflow is:

```text
Ansible Control Machine
          │
          ▼
      ec2_key
          │
          ▼
     AWS Key Pair
          │
          │ creation result
          ▼
       register
          │
          ▼
      keyout
          │
          ▼
  Private Key Returned
          │
          ▼
     copy module
          │
          ▼
   Local .pem file
```

The private key is returned when the key pair is created, so the result must be captured immediately when it changes.

The follow-up task is therefore gated with:

```text
when: keyout.changed
```

This creates a reusable pattern:

```text
Create
  ↓
Register
  ↓
Conditionally process result
```

---

## 16. AWS EC2 Provisioning Flow

The EC2 provisioning architecture is:

```text
Ansible Control Machine
          │
          │ AWS API
          ▼
      ec2_instance
          │
          ├── AMI
          ├── Instance Type
          ├── Key Pair
          ├── Security Group
          ├── Region
          └── exact_count
                    │
                    ▼
               EC2 Instance
```

The practical uses:

```text
exact_count: 1
```

to make repeated execution converge on the requested instance count rather than continually creating additional instances.

---

## 17. AWS Authentication Boundary

AWS credentials are separate from the Ansible playbook.

The demonstrated model is:

```text
Environment Variables
        │
        ▼
AWS Authentication
        │
        ▼
Ansible AWS Module
        │
        ▼
AWS API
```

The practical explicitly warns against putting access keys directly into playbooks or committing them to Git repositories.

The intended repository boundary is:

```text
Git Repository
    ≠
AWS Access Keys
    ≠
Private SSH Keys
```

---

## 18. Complete Architecture

The complete project can therefore be represented as:

```text
                         ┌──────────────────────┐
                         │   Ansible Control    │
                         │       Machine        │
                         │                      │
                         │  ansible.cfg         │
                         │  Inventory           │
                         │  Playbooks           │
                         │  Variables           │
                         │  Templates           │
                         │  Roles               │
                         └──────────┬───────────┘
                                    │
                       ┌────────────┴────────────┐
                       │                         │
                    SSH Path                 API Path
                       │                         │
                       ▼                         ▼
             ┌──────────────────┐       ┌──────────────────┐
             │ Managed Linux    │       │       AWS        │
             │ Hosts            │       │                  │
             │                  │       │  EC2 Key Pair    │
             │ web01            │       │  EC2 Instance    │
             │ web02            │       │                  │
             │ db01             │       └──────────────────┘
             └────────┬─────────┘
                      │
                      ▼
             Desired Configuration
                      │
          ┌───────────┼────────────┐
          │           │            │
       Packages     Services     Files
          │           │            │
          └───────────┼────────────┘
                      │
              ┌───────┴────────┐
              │                │
          Templates         Handlers
              │                │
              └───────┬────────┘
                      │
                   Roles
```

---

## 19. Architecture Decisions

### Decision 1 — Use a Dedicated Control Machine

Ansible is installed on the control machine rather than on every managed host.

**Reason:** This follows Ansible's agentless architecture and keeps the execution engine centralized.

### Decision 2 — Use Repository-Level Inventory

The inventory belongs with the automation project rather than relying on the global inventory.

**Reason:** The target definition becomes portable with the project.

### Decision 3 — Separate Configuration Data from Automation Logic

Variables are separated from playbook task definitions where practical.

**Reason:** Changing environment/application values should not require rewriting the automation logic.

### Decision 4 — Use Facts for Host-Aware Automation

Runtime host information is used to make provisioning decisions.

**Reason:** The same automation can adapt to different target characteristics.

### Decision 5 — Use Templates for Dynamic Configuration

Jinja2 templates are used when configuration contains variables.

**Reason:** Static configuration and changing values can be separated.

### Decision 6 — Use Handlers for Dependent Actions

Service restarts are triggered through notifications rather than unconditional tasks.

**Reason:** Avoid unnecessary service disruption when configuration has not changed.

### Decision 7 — Use Roles for Reusable Automation

Related tasks, handlers, templates, files and variables are organized into a standardized role.

**Reason:** Reduce playbook complexity and create reusable automation boundaries.

### Decision 8 — Use a Separate API Execution Model for AWS

AWS provisioning is executed from the control machine through AWS APIs rather than through SSH.

**Reason:** AWS is an API target rather than an operating-system target.

### Decision 9 — Use `exact_count` for EC2 Convergence

The EC2 automation specifies the desired instance count.

**Reason:** Repeated execution should converge on the requested count rather than continually creating new instances.

### Decision 10 — Keep Cloud Provisioning Scope Explicit

The AWS portion demonstrates that Ansible can provision cloud resources, while the course material positions Terraform as the more specialized tool for heavy cloud infrastructure management.

This project therefore does not claim to be a complete cloud infrastructure-as-code platform.

---

## 20. Architecture Boundaries

This architecture demonstrates:

- Agentless Linux configuration management.
- Inventory-driven targeting.
- SSH-based remote execution.
- Declarative configuration.
- Variables and facts.
- Conditional execution.
- Iterative task execution.
- Dynamic configuration.
- Change-triggered handlers.
- Reusable roles.
- AWS API-based EC2 provisioning.

It does **not** establish:

- A production Ansible Automation Controller/AWX architecture.
- Enterprise secrets management.
- CI/CD architecture.
- Terraform-based infrastructure provisioning.
- Kubernetes orchestration.
- Production monitoring architecture.
- Multi-account AWS governance.
- Disaster-recovery architecture.
- Production-scale high availability.

These boundaries are intentionally preserved so that the architecture represents the demonstrated practical rather than implying capabilities that were not established.
