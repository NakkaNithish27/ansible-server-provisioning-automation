# Implementation

[← Back to README](../README.md) | [Architecture](architecture.md) | [Validation](validation.md) | [Limitations & Future Work](limitations-and-future-work.md)

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/768f8962-0a68-42b7-961f-57880649aabe" />

---

## 1. Implementation Overview

The implementation was developed incrementally rather than starting with a large role-based structure.

The practical progression was:

```text
Ansible Setup
      ↓
Inventory & SSH Connectivity
      ↓
Multi-Host Inventory
      ↓
Ad Hoc Commands
      ↓
Playbooks & Modules
      ↓
Configuration
      ↓
Variables & Debug
      ↓
Facts
      ↓
Conditional Provisioning
      ↓
Loops
      ↓
Templates
      ↓
Handlers
      ↓
Roles
      ↓
AWS API Automation
```

This progression allowed each new abstraction to build on the previous implementation.

---

## 2. Ansible Environment Setup

The implementation begins with an Ansible control machine and Linux hosts that are reachable through SSH.

The control machine contains the Ansible project files:

```text
ansible/
├── ansible.cfg
├── inventory
└── playbooks/
```

The managed Linux hosts do not require an Ansible service. Ansible uses its agentless execution model and connects to the hosts when automation is executed.

The project configuration is kept with the automation rather than relying exclusively on the system-wide Ansible configuration.

---

## 3. Inventory and SSH Connectivity

### 3.1 Initial Host Configuration

The first implementation step is defining the target host and its SSH connection information in the inventory.

The inventory represents:

```text
Host
 ├── Address
 ├── SSH user
 └── SSH key
```

This establishes the basic relationship:

```text
Ansible Control Machine
          │
          │ SSH
          ▼
     Managed Host
```

The inventory is therefore the source of the target and connection information used by Ansible.

---

### 3.2 Connectivity Validation

Connectivity is tested before building more complex automation.

The basic validation uses the Ansible ping module:

```bash
ansible <host> -m ping
```

The objective is to verify:

```text
Inventory
    ↓
SSH credentials
    ↓
Network connectivity
    ↓
Ansible execution
```

are working.

---

## 4. Multi-Host Inventory

The inventory was then expanded from a single host to multiple hosts.

The practical adds:

```text
web01
web02
db01
```

and organizes them into groups:

```text
dc_oregon
├── webservers
│   ├── web01
│   └── web02
│
└── dbservers
    └── db01
```

This creates multiple targeting levels:

```text
Individual Host
     ↓
Functional Group
     ↓
Parent Group
     ↓
All Hosts
```

The implementation also moves common SSH settings into group-level variables rather than repeating them for every host.

This demonstrates the DRY principle and inventory inheritance. Host-level values take precedence when a host needs a different value.

---

## 5. Ad Hoc Automation

After connectivity was established, Ansible modules were used directly through ad hoc commands.

For example, package installation uses the `yum` module:

```bash
ansible webservers   -m ansible.builtin.yum   -a "name=httpd state=present"   -i inventory   --become
```

Service management uses:

```bash
ansible webservers   -m ansible.builtin.service   -a "name=httpd state=started enabled=yes"   -i inventory   --become
```

The implementation demonstrates that the same module can be used to establish a desired state without manually executing the equivalent operating-system commands.

The package and service operations also demonstrate idempotency:

```text
First run
    ↓
changed

Second run
    ↓
ok
```

---

## 6. File Deployment

The `copy` module was used to transfer a locally created file to the managed web servers.

Conceptually:

```text
Control Machine
      │
      │ copy
      ▼
/var/www/html/index.html
```

The implementation verifies that repeating the same operation does not continually modify the target.

A source-file modification causes Ansible to detect the difference and transfer the updated file.

This establishes an important implementation pattern:

```text
Source State
     ↓
Compare
     ↓
Target State
     ↓
Change only when required
```

---

## 7. Playbook Conversion

The next implementation stage converts individual ad hoc operations into structured YAML playbooks.

The playbook model is:

```text
Playbook
   ↓
Play
   ↓
Target Hosts
   ↓
Tasks
   ↓
Modules
```

A playbook describes **what should happen** and **where it should happen**.

A task invokes an Ansible module to perform an individual operation.

A playbook execution is performed with:

```bash
ansible-playbook -i <inventory-path> <playbook>.yaml
```

The practical shows the first execution producing `changed` results and subsequent execution producing `ok` where the desired state is already satisfied.

---

## 8. Module Selection and Troubleshooting

The implementation uses Ansible modules rather than relying on shell commands for standard configuration-management operations.

The modules demonstrated include:

```text
ansible.builtin.yum
ansible.builtin.apt
ansible.builtin.service
ansible.builtin.copy
ansible.builtin.file
ansible.builtin.user
template
amazon.aws.ec2_key
amazon.aws.ec2_instance
```

The implementation approach is to use Ansible documentation to discover:

- Module purpose
- Requirements
- Parameters
- Examples

The goal is to understand how to find and use the documentation rather than memorize the entire module ecosystem.

---

## 9. Project Configuration

Project-level Ansible behavior is represented by:

```text
ansible.cfg
```

The implementation uses project configuration to avoid depending entirely on system-wide defaults.

The configuration can define behavior such as:

- Inventory location
- Host-key checking
- Parallel execution
- Logging

The project-level `./ansible.cfg` provides predictable behavior when running the repository.

---

## 10. Variables and Parameterization

The implementation then replaces hardcoded values with variables.

For example:

```yaml
vars:
  dbname: electric
  dbuser: current
  dbpass: tesla
```

Tasks reference the variables rather than embedding the values directly.

The resulting structure is:

```text
Task Logic
    │
    └── references
            │
            ▼
        Variables
```

The implementation demonstrates that changing a value does not require rewriting the task definition.

---

### 10.1 Debugging Variables

The `debug` module is used to inspect values during execution.

The flow is:

```text
Variable
   ↓
debug
   ↓
Execution Output
```

This provides an inspection mechanism similar to printing a variable during normal programming.

Verbose execution can also expose more detailed module information:

```bash
ansible-playbook ... -vv
```

---

### 10.2 Registered Task Results

Task output can also be captured using a registered variable.

The pattern is:

```text
Task
  ↓
register
  ↓
Registered Result
  ↓
debug / condition / later task
```

This becomes particularly important later in the AWS automation where the result of an AWS resource operation controls a subsequent task.

---

## 11. Facts and Runtime Discovery

Ansible automatically gathers facts at the beginning of a play unless fact gathering is disabled.

The implementation uses the `setup` module and fact variables to inspect host characteristics.

The conceptual flow is:

```text
Target Host
    ↓
setup module
    ↓
Runtime Facts
    ↓
Playbook Logic
```

Facts include information about the operating system and other system properties.

The implementation also demonstrates discovering the structure of facts directly:

```bash
ansible <host> -m setup
```

This is useful when the exact fact variable name or nested structure is unknown.

---

## 12. Multi-OS Provisioning

The provisioning series introduces CentOS and Ubuntu targets.

The implementation needs to account for differences such as:

| Requirement | CentOS | Ubuntu |
|---|---|---|
| Package module | `yum` | `apt` |
| NTP package | `chrony` | `ntp` |
| Service | `chronyd` | `ntp` |
| Configuration | `/etc/chrony.conf` | `/etc/ntp.conf` |

The practical uses separate tasks with `when` conditions so each task only executes on the appropriate operating system.

The implementation pattern is:

```text
Playbook runs against all hosts
          │
          ├── CentOS task
          │      ↓
          │   when == CentOS
          │
          └── Ubuntu task
                 ↓
              when == Ubuntu
```

This allows one playbook to manage heterogeneous hosts without attempting incompatible package or service operations.

---

## 13. Loop-Based Provisioning

The next implementation step removes repeated package-installation tasks.

Instead of:

```text
Install chrony
Install wget
Install git
Install zip
Install unzip
```

the implementation uses one task:

```yaml
loop:
  - chrony
  - wget
  - git
  - zip
  - unzip
```

and references the current value through:

```text
{{ item }}
```

The same pattern is applied to the Ubuntu package task with its appropriate package list.

The resulting abstraction is:

```text
One Task
   +
Collection
   ↓
Repeated Execution
```

This reduces duplication and makes adding another package a data change rather than a structural playbook change.

---

## 14. Static and Dynamic File Management

The implementation distinguishes between static file deployment and dynamic configuration.

### Static content

The `copy` module is appropriate when the content should be transferred as-is.

### Dynamic configuration

The `template` module is used when the source contains Jinja2 variables.

The workflow is:

```text
Existing target configuration
          ↓
Extract
          ↓
Store template locally
          ↓
Replace changing values
          ↓
Add Jinja2 variables
          ↓
Define variables
          ↓
Render template
          ↓
Deploy configuration
```

The implementation demonstrates extracting NTP configuration files, placing them under `templates/`, replacing static NTP server values with variables, and deploying the rendered configuration.

---

## 15. Configuration Variables

The dynamic configuration uses variables such as:

```text
ntp0
ntp1
ntp2
ntp3
```

The values are maintained separately from the template.

The resulting relationship is:

```text
group_vars/all
      │
      │ variables
      ▼
Jinja2 Template
      │
      │ rendered
      ▼
Target Configuration
```

This means a change to the NTP server values can be made in one variable location instead of editing every configuration file.

The implementation also demonstrates the `backup: yes` option for protecting an existing configuration before replacement.

---

## 16. Handler-Based Service Management

The template implementation exposes an important problem.

An unconditional restart task would execute even when the configuration did not change.

The implementation therefore introduces a handler:

```text
Template Task
     │
     ├── unchanged → no notification
     │
     └── changed
            ↓
         notify
            ↓
         Handler
            ↓
       Restart Service
```

The handler is therefore tied to the change state of the configuration task.

The implementation demonstrates that a handler runs when the associated template changes and does not run when the template remains unchanged.

This establishes the implementation pattern:

> **Change configuration → notify dependent action only when required.**

Handlers are not limited to service restarts; they can execute other tasks when triggered.

---

## 17. Role Conversion

After the playbook has accumulated:

- Tasks
- Variables
- Templates
- Files
- Handlers

the implementation converts it into a reusable role.

The role is named:

```text
post-install
```

The standardized structure is:

```text
roles/
└── post-install/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── defaults/
    │   └── main.yml
    ├── templates/
    └── files/
```

The source material also describes `vars/main.yml` as part of the standard role structure; this repository's minimal skeleton intentionally does not include it until role variables are actually required.

---

### 17.1 Role Entry Point

The most important role file is:

```text
tasks/main.yml
```

When the role is executed, Ansible starts with this task file.

Role-local resources are resolved according to the standardized directory structure.

For example:

```text
template task
     ↓
templates/
     ↓
template file
```

and:

```text
copy task
     ↓
files/
     ↓
static file
```

The role structure therefore removes the need to hardcode paths such as `templates/...` when referencing role-local resources.

---

## 18. Simplifying the Playbook

Before role conversion:

```text
Playbook
├── variables
├── tasks
├── handlers
├── template references
└── file references
```

After role conversion:

```text
Playbook
└── roles:
      └── post-install
```

The implementation moves the detailed configuration logic into the role.

This creates a clearer separation between:

```text
Playbook
    =
Which role should run?

Role
    =
How should the server be configured?
```

---

## 19. Role Defaults and Overrides

Roles separate reusable logic from environment-specific values.

The demonstrated model is:

```text
Role defaults
      ↓
Environment/project values
      ↓
Override when necessary
```

For example:

```text
defaults/main.yml
    ntp0 = generic value

       ↓ override

playbook/project value
    ntp0 = environment-specific value

       ↓

template rendering
       ↓
configuration change
       ↓
handler
```

This allows the role logic to remain unchanged while different environments supply different values.

---

## 20. Community Roles and Ansible Galaxy

The practical also demonstrates obtaining a community role from Ansible Galaxy.

The purpose is primarily to understand:

- Role structure
- Expert organization patterns
- `include_tasks`
- `include_vars`
- OS-specific role organization

The community role is not treated as personally authored project code.

The role is used as reference material for understanding how reusable automation can be organized.

---

## 21. AWS Automation

The final implementation stage changes from Linux configuration management to AWS API automation.

The AWS playbook targets the control machine:

```yaml
hosts: localhost
gather_facts: false
```

because the AWS modules communicate with AWS APIs rather than an operating-system target over SSH.

The AWS implementation demonstrates:

```text
AWS Authentication
       ↓
AWS Collection
       ↓
AWS Module
       ↓
AWS API
       ↓
AWS Resource
```

---

## 22. AWS Key-Pair Creation

The first AWS operation creates an EC2 key pair using the AWS collection.

The result is captured with `register`:

```text
ec2_key
   ↓
register
   ↓
keyout
```

When the key is newly created, the generated private key is available in the returned result.

A subsequent `copy` task writes the private key to a local file only when the key-creation task reports a change:

```text
keyout.changed
       ↓
    true?
     /     yes   no
    │     │
  copy   skip
```

This combines several concepts previously developed in the project:

```text
Module
+
register
+
changed
+
when
+
copy
```

---

## 23. AWS EC2 Instance Provisioning

The next step uses:

```text
amazon.aws.ec2_instance
```

to launch an EC2 instance.

The implementation provides the required EC2 parameters, including values such as:

- AMI
- Instance type
- Key pair
- Security group
- Region
- Desired instance count

The practical uses:

```text
exact_count: 1
```

so repeated execution converges on the requested number of instances rather than creating an additional instance on every run.

The implementation therefore applies the same desired-state principle used earlier for Linux resources to cloud resource provisioning.

---

## 24. AWS Credential Handling

AWS credentials are treated as environment-specific authentication data rather than playbook configuration.

The implementation does **not** place credentials directly into the repository.

The intended separation is:

```text
Playbook
    ↓
AWS module
    ↓
AWS credential provider
    ↓
AWS API
```

Private keys, access keys, tokens and other secrets must remain outside version-controlled project files.

---

## 25. Troubleshooting Approach

Troubleshooting is performed progressively rather than immediately changing the automation.

### SSH authentication failure

```text
Permission denied
      ↓
Check SSH user
      ↓
Check host-specific variable
      ↓
Correct more-specific value
```

The Ubuntu host example demonstrates how an incorrect global SSH user can be overridden with a host-specific value.

### Undefined variable

```text
variable undefined
      ↓
Check variable name
      ↓
Check variable scope
      ↓
Check spelling
```

### Module failure

```text
Module error
      ↓
Read error
      ↓
Check module requirements
      ↓
Check parameters
      ↓
Check target state
```

The module documentation workflow is part of the practical troubleshooting approach.

---

## 26. Verbose Troubleshooting

When normal output is insufficient, verbosity is increased.

```bash
ansible-playbook ... -v
ansible-playbook ... -vv
ansible-playbook ... -vvv
ansible-playbook ... -vvvv
```

The progression is:

```text
-v
  ↓
Additional module output

-vv
  ↓
Configuration/version/playbook details

-vvv
  ↓
Connection details

-vvvv
  ↓
Maximum verbosity
```

The approach is to start with lower verbosity and increase it when deeper execution or connection information is required.

---

## 27. Implementation Pattern

The complete implementation can be reduced to:

```text
1. Define targets
        ↓
2. Verify connectivity
        ↓
3. Perform operation manually with modules
        ↓
4. Convert operation into playbook
        ↓
5. Parameterize values
        ↓
6. Discover runtime facts
        ↓
7. Add conditional execution
        ↓
8. Remove repetition with loops
        ↓
9. Externalize dynamic configuration
        ↓
10. Trigger dependent actions with handlers
        ↓
11. Refactor into reusable role
        ↓
12. Extend automation to AWS APIs
```

This is the central implementation progression of the project.

---

## 28. Engineering Principles Applied

The implementation demonstrates several reusable engineering patterns:

| Principle | Implementation |
|---|---|
| Desired state | Package/service/file states |
| Idempotency | `changed` on first application, `ok` when state already matches |
| DRY | Inventory groups, variables, loops |
| Separation of concerns | Tasks, variables, templates, handlers, roles |
| Parameterization | Variables and facts |
| Runtime discovery | `setup` / gathered facts |
| Conditional execution | `when` |
| Iteration | `loop` |
| Dynamic configuration | Jinja2 templates |
| Event-driven execution | Handlers |
| Reusability | Roles |
| API automation | AWS modules |
| Result-driven control flow | `register` + `when` |
| Cloud convergence | `exact_count` |
| Troubleshooting | Debug module and verbosity |

---

## 29. Implementation Boundary

The implementation demonstrates practical Ansible configuration management and AWS API automation.

It does not document or claim:

- A production Ansible Automation Controller deployment.
- Enterprise-grade secrets management.
- CI/CD integration.
- Terraform infrastructure provisioning.
- Kubernetes automation.
- Production monitoring.
- Enterprise AWS governance.
- Multi-account infrastructure management.
- Production disaster recovery.

The implementation should therefore be understood as a **practical automation project demonstrating core Ansible engineering patterns**, rather than as a complete enterprise automation platform.
