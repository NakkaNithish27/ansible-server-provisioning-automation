# Validation

[← Back to README](../README.md) | [Architecture](architecture.md) | [Implementation](implementation.md) | [Limitations & Future Work](limitations-and-future-work.md)

---

## 1. Validation Overview

Validation for this project focuses on proving that the Ansible automation behaves as intended across the major implementation stages:

```text
Connectivity
    ↓
Targeting
    ↓
Playbook Execution
    ↓
Idempotency
    ↓
Variables & Facts
    ↓
Conditional Execution
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

The goal is not to collect screenshots of every command.

The goal is to establish a small set of high-signal results that demonstrate:

- Ansible can reach the intended targets.
- The automation performs the requested configuration.
- Repeated execution converges on the desired state.
- Runtime facts and conditions select the correct behavior.
- Dynamic configuration is rendered correctly.
- Handlers execute only when relevant changes occur.
- Role-based automation executes successfully.
- AWS resources can be managed through Ansible modules.

The learning material explicitly demonstrates `--syntax-check`, check mode (`-C`), idempotent `changed`/`ok` behavior, and verbosity-based troubleshooting as validation mechanisms.

---

## 2. Validation Philosophy

The validation model follows:

```text
Claim
  ↓
Test
  ↓
Expected Result
  ↓
Observed Result
  ↓
Evidence
```

For example:

```text
Claim:
Playbook is idempotent

        ↓

Test:
Run the same playbook twice

        ↓

Expected:
First run → changed
Second run → ok

        ↓

Evidence:
Playbook output
```

This distinction is important because documentation explains what the automation is intended to do, while execution evidence demonstrates what actually happened.

The repository should only label an execution result as personal evidence when it comes from the completed environment. The supplied learning material establishes the practical capability but should not itself be presented as proof of personal execution.

---

# 3. Validation Levels

Validation is divided into four levels:

```text
Level 1
Static Validation
    ↓
Level 2
Execution Validation
    ↓
Level 3
Behavior Validation
    ↓
Level 4
Resource Validation
```

### Level 1 — Static Validation

Checks that the playbook can be parsed.

### Level 2 — Execution Validation

Checks that Ansible can connect and execute the automation.

### Level 3 — Behavior Validation

Checks that the target reaches the intended state and repeated execution behaves correctly.

### Level 4 — Resource Validation

Checks externally observable resources such as AWS EC2 resources.

---

# 4. Static Playbook Validation

Before executing a playbook against infrastructure, syntax validation should be performed.

Use:

```bash
ansible-playbook -i <inventory> <playbook>.yml --syntax-check
```

### Expected result

The playbook should pass YAML/playbook syntax validation without executing tasks against the targets.

The supplied material specifically identifies `--syntax-check` as a pre-execution validation mechanism.

### What this proves

```text
YAML structure
      +
Playbook syntax
      ↓
Valid enough for execution
```

### What this does NOT prove

A successful syntax check does **not** prove:

- SSH connectivity
- Correct credentials
- Correct module parameters
- Correct target state
- Successful package installation
- Successful service configuration
- Correct runtime variables

Syntax validation is therefore only the first validation layer.

---

# 5. Dry-Run / Check-Mode Validation

Before making changes, check mode can be used:

```bash
ansible-playbook -i <inventory> <playbook>.yml -C
```

or:

```bash
ansible-playbook -i <inventory> <playbook>.yml --check
```

### Purpose

Check mode attempts to simulate execution without applying the changes.

The practical material describes this as a useful pre-execution practice while explicitly warning that a successful dry run is **not a guarantee** that the real execution will succeed.

### Validation principle

```text
Syntax Check
     ↓
Check Mode
     ↓
Actual Execution
```

Check mode should therefore complement, not replace, real execution.

---

# 6. Connectivity Validation

Connectivity is the first runtime validation.

Use:

```bash
ansible <host> -m ping
```

or target a group:

```bash
ansible <group> -m ping
```

### Expected result

The intended targets should respond successfully.

Conceptually:

```text
Inventory
    ↓
SSH User
    ↓
SSH Key
    ↓
Network Connectivity
    ↓
Ansible
    ↓
Target Responds
```

The project architecture uses SSH between the control machine and Linux managed hosts.

### What this proves

- Inventory target is reachable.
- SSH configuration is usable.
- Ansible can communicate with the managed host.
- The basic control-to-target path works.

---

# 7. Host and Group Targeting Validation

The inventory contains individual hosts and groups.

Validation should confirm that the intended targeting scope is respected.

Examples:

```bash
ansible web01 -m ping
```

```bash
ansible webservers -m ping
```

```bash
ansible dc_oregon -m ping
```

### Expected behavior

```text
web01
   ↓
Only web01

webservers
   ↓
web01 + web02

dc_oregon
   ↓
All hosts belonging to the parent group
```

This validates the inventory hierarchy rather than merely proving that one host is reachable.

---

# 8. Ad Hoc Module Validation

The initial configuration-management operations should be validated using the corresponding modules.

Examples include:

```text
yum / apt
service
copy
file
user
```

The expected Ansible result states are:

```text
ok
changed
failed
```

The important distinction is:

```text
changed
    =
Ansible modified the target

ok
    =
Target already satisfied the requested state

failed
    =
Requested state could not be achieved
```

The practical material uses this `changed` versus `ok` behavior as the observable indication of idempotency.

---

# 9. Idempotency Validation

Idempotency is one of the highest-value validation areas in the project.

The test is simple:

```text
Run playbook
     ↓
Record result
     ↓
Run same playbook again
     ↓
Compare result
```

### Expected behavior

First execution:

```text
changed
```

Second execution:

```text
ok
```

for resources that are already in the desired state.

The supplied material explicitly demonstrates that the first playbook execution can report `changed`, while the second execution reports `ok` because the packages and services are already configured.

### Why this matters

It demonstrates:

```text
Desired State
      ↓
Current State
      ↓
Difference?
   /        No         Yes
 │           │
ok        changed
```

This is stronger evidence than simply showing that a playbook ran successfully once.

---

# 10. Variable Validation

Variables are validated by confirming that:

1. The variable is defined.
2. The variable is resolved during execution.
3. The module receives the expected value.
4. Changing the variable changes the resulting behavior where appropriate.

The `debug` module can be used for direct inspection.

Conceptually:

```text
Variable
    ↓
debug
    ↓
Expected Value
```

Verbose execution can also be used:

```bash
ansible-playbook -i <inventory> <playbook>.yml -vv
```

The learning material specifically uses `-vv` to inspect the values substituted into module parameters.

---

# 11. Registered Variable Validation

Registered task results are validated by confirming that:

```text
Task
  ↓
register
  ↓
Result Variable
  ↓
debug / condition / later task
```

contains the expected information.

This becomes particularly important in the AWS implementation, where the result of `ec2_key` is registered and then used by a conditional follow-up task.

---

# 12. Facts Validation

Facts are validated by gathering host information:

```bash
ansible <host> -m setup
```

### Expected result

The response should contain runtime information about the target, including properties such as:

- Operating system
- Architecture
- Kernel
- CPU
- Network information

The practical explains that the `setup` module collects target information at runtime and that this information can be used by conditions and templates.

### What this proves

```text
Target
   ↓
Fact Gathering
   ↓
Runtime Information
   ↓
Playbook Decision
```

---

# 13. Conditional Execution Validation

The multi-OS provisioning logic uses conditions to select different tasks for different operating systems.

The validation should confirm:

```text
CentOS target
    ↓
CentOS-specific task executes

Ubuntu target
    ↓
Ubuntu-specific task executes
```

and that incompatible tasks are skipped.

Conceptually:

```text
                 Provisioning
                      │
             ┌────────┴────────┐
             │                 │
          CentOS             Ubuntu
             │                 │
            yum               apt
             │                 │
          chrony               ntp
```

The purpose is to verify that runtime facts are influencing task selection rather than merely being gathered.

---

# 14. Loop Validation

Loop-based package provisioning is validated by checking that every expected item is processed.

For example:

```text
chrony
wget
git
zip
unzip
```

### Expected behavior

One task should process the collection rather than requiring five duplicated tasks.

Validation should confirm:

```text
loop
  ↓
item 1 → processed
item 2 → processed
item 3 → processed
item 4 → processed
item 5 → processed
```

A subsequent playbook run should again demonstrate idempotent behavior for already-installed packages.

---

# 15. Template Validation

Template validation has two levels:

### 15.1 Rendering

Confirm that variables are correctly substituted into the generated configuration.

```text
Variables
    ↓
Jinja2 Template
    ↓
Rendered Configuration
```

### 15.2 Target Configuration

Confirm that the rendered configuration reaches the intended location on the target.

For example:

```text
Template
   ↓
/etc/chrony.conf
```

or:

```text
Template
   ↓
/etc/ntp.conf
```

The practical demonstrates replacing hardcoded NTP server values with Jinja2 variables and deploying the rendered configuration.

---

# 16. Template Change Validation

A particularly important validation is to modify a template variable and observe the resulting change.

Test:

```text
Initial variable
      ↓
Run playbook
      ↓
Configuration deployed

Change variable
      ↓
Run playbook
      ↓
Configuration changes
```

This proves that:

```text
Variable
   ↓
Template
   ↓
Configuration
```

is functioning as intended.

It also provides the setup required to validate handler behavior.

---

# 17. Handler Validation

Handler behavior should be tested using two executions.

### Test 1 — Configuration unchanged

```text
Run playbook
     ↓
Template unchanged
     ↓
No notification
     ↓
Handler does not run
```

### Test 2 — Configuration changed

```text
Modify variable/template
     ↓
Run playbook
     ↓
Template changed
     ↓
notify
     ↓
Handler runs
```

The practical specifically demonstrates handlers as change-triggered actions and shows that the handler should execute when the template changes rather than on every run.

This is one of the strongest behavioral validation points in the project.

---

# 18. Role Validation

After converting the automation into the `post-install` role, validation should confirm that the role performs the same required configuration as the previous playbook.

The validation flow is:

```text
Playbook
   ↓
post-install role
   ↓
Tasks
   ↓
Templates
   ↓
Handlers
   ↓
Target State
```

### Expected result

The role should successfully:

- Execute its tasks.
- Resolve role-local templates.
- Use role defaults/variables where applicable.
- Trigger its handlers when relevant.
- Produce the expected target configuration.

The role's `tasks/main.yml` acts as the entry point, with role-local templates and files resolved through the role structure.

---

# 19. Role Idempotency Validation

The role should also be executed twice.

```text
First role execution
        ↓
changed where configuration is applied

Second role execution
        ↓
ok where state already matches
```

This confirms that refactoring the playbook into a role did not remove the underlying desired-state behavior.

---

# 20. AWS Dependency Validation

The AWS section introduces additional dependencies.

The practical identifies the dependency chain as:

```text
pip
 ↓
boto3
 ↓
amazon.aws collection
 ↓
AWS modules
```

Validation should confirm that the required dependencies are available before attempting AWS resource operations.

---

# 21. AWS Authentication Validation

AWS authentication should be tested without embedding credentials into the playbook.

The expected model is:

```text
Environment Variables
        ↓
AWS Authentication
        ↓
Ansible AWS Module
        ↓
AWS API
```

The learning material explicitly states that access keys should not be placed in playbook source or committed to Git.

### Validation requirement

A successful AWS API operation demonstrates that:

```text
Credentials
+
Region
+
AWS permissions
+
AWS module
```

are functioning together.

---

# 22. AWS Key-Pair Validation

The key-pair workflow should be validated in stages:

```text
ec2_key
   ↓
AWS key pair created
   ↓
register result
   ↓
private key returned
   ↓
copy task
   ↓
local key file
```

The follow-up copy operation should only execute when:

```text
keyout.changed
```

This validates the reusable:

```text
register → changed → when
```

pattern.

### Important security boundary

The resulting private key is **execution data**, not a repository artifact.

It must not be committed.

---

# 23. EC2 Provisioning Validation

The EC2 automation should be validated by inspecting AWS after playbook execution.

The expected flow is:

```text
Ansible
   ↓
ec2_instance
   ↓
AWS API
   ↓
EC2 instance
   ↓
AWS Console / API inspection
```

The practical uses:

```text
exact_count: 1
```

to make repeated execution converge on one desired instance rather than continually creating new instances.

---

# 24. EC2 Idempotency Validation

Run the AWS playbook more than once.

### First execution

Expected:

```text
EC2 instance created
```

### Subsequent execution

Expected:

```text
Desired count already satisfied
```

and no unnecessary additional instance should be created.

The test therefore validates:

```text
Desired count = 1
       ↓
Current count = 1
       ↓
No duplicate resource
```

This is the AWS equivalent of the desired-state validation used earlier for Linux configuration.

---

# 25. AWS Region Validation

AMI identifiers are region-specific in the demonstrated AWS workflow.

Therefore the validation should confirm:

```text
Playbook Region
      =
AMI Region
```

If the AMI does not exist in the selected region, the AWS operation can fail with an AMI-not-found type of error.

The learning material explicitly identifies region mismatch as an encountered AWS failure mode.

---

# 26. AWS Cleanup Validation

After completing the AWS practical, temporary resources should be removed where they are no longer required.

The cleanup sequence is:

```text
Terminate EC2 instance
        ↓
Delete EC2 key pair
        ↓
Delete local sample.pem
        ↓
Revoke/delete temporary credentials if applicable
```

Cleanup is operational hygiene rather than evidence of a project capability.

---

# 27. Troubleshooting Validation

Validation also includes the ability to identify why an automation attempt failed.

The practical distinguishes between:

```text
Syntax errors
    ↓
Usually straightforward

Logical errors
    ↓
Require deeper investigation
```

Examples of logical errors include:

- Wrong SSH user.
- Wrong SSH key.
- Missing sudo privileges.
- Incorrect module parameters.
- Missing dependencies.
- Incorrect AWS region.
- Missing AWS collection.
- Duplicate EC2 resources caused by missing `exact_count`.

The material recommends increasing verbosity when deeper execution information is required.

---

# 28. Verbosity-Based Validation

Use verbosity progressively:

```bash
ansible-playbook ... -v
```

```bash
ansible-playbook ... -vv
```

```bash
ansible-playbook ... -vvv
```

```bash
ansible-playbook ... -vvvv
```

The progression is:

```text
-v
   ↓
Additional module information

-vv
   ↓
Configuration/version/playbook details

-vvv
   ↓
Connection information

-vvvv
   ↓
Maximum detail
```

At higher verbosity, connection details such as the user and identity file can help diagnose authentication problems.

---

# 29. Evidence Strategy

The repository should contain **high-signal evidence**, not screenshots of every command.

The preferred evidence structure is:

```text
evidence/
└── screenshots/
```

Only evidence from the completed personal environment should be presented as proof of execution.

Course screenshots and lecture material should not be used as personal project evidence.

---

# 30. Recommended Evidence

## Evidence 1 — Successful Connectivity

Show a successful Ansible ping against the intended host or group.

### Proves

```text
Inventory
+
SSH
+
Ansible connectivity
```

### Interview question supported

> How does Ansible connect to the managed nodes?

---

## Evidence 2 — Idempotent Playbook Execution

Prefer evidence showing:

```text
First run → changed
Second run → ok
```

### Proves

- Desired-state behavior.
- Idempotency.
- Repeatable automation.

### Interview question supported

> How did you verify that your playbook was idempotent?

---

## Evidence 3 — Template + Handler

Show a configuration change that causes:

```text
Template changed
      ↓
Handler notified
      ↓
Service action
```

### Proves

- Dynamic configuration.
- Change detection.
- Event-driven handler execution.

### Interview question supported

> Why did you use a handler instead of restarting the service every time?

---

## Evidence 4 — Role Execution

Show the role structure or successful role execution.

### Proves

- Modular automation.
- Role organization.
- Reusable configuration management.

### Interview question supported

> Why did you convert the playbook into a role?

---

## Evidence 5 — AWS Provisioning

Show the successful AWS resource created through the Ansible playbook.

### Proves

```text
Ansible
   ↓
AWS API
   ↓
EC2
```

### Interview question supported

> How did Ansible interact with AWS?

---

# 31. Evidence Mapping

| Project Claim | Validation | Strongest Evidence |
|---|---|---|
| Ansible can reach managed hosts | `ansible <host> -m ping` | Successful ping output |
| Inventory targeting works | Host/group ping | Targeted output |
| Playbook executes | Playbook run | Play recap |
| Automation is idempotent | Two consecutive runs | `changed` → `ok` |
| Variables resolve correctly | `debug` / `-vv` | Variable output |
| Facts are available | `setup` | Fact output |
| Conditions select correct tasks | Multi-OS playbook | Task/skip output |
| Loops process expected items | Loop execution | Task results |
| Templates render configuration | Variable/template change | Rendered target state |
| Handlers respond to changes | Template change | Handler execution |
| Role works | Role execution | Play recap / target state |
| AWS key pair automation works | `ec2_key` execution | AWS key pair + play output |
| AWS EC2 automation works | `ec2_instance` execution | EC2 resource |
| EC2 automation converges | Repeat execution | No duplicate instance |
| Troubleshooting workflow works | Controlled failure/recovery | Sanitized terminal evidence |

Not every row needs a screenshot in the final repository.

The purpose is to retain enough evidence to support the major project claims.

---

# 32. Evidence Quality Rules

Good evidence should:

- Show the relevant command or result.
- Be readable.
- Provide enough context to understand what happened.
- Avoid exposing credentials.
- Avoid exposing private keys.
- Avoid unnecessary environment-specific information.
- Avoid unrelated terminal history.
- Avoid duplicating the same result multiple times.

### Sanitize

Before publishing evidence, inspect screenshots for:

```text
AWS access keys
AWS secret keys
SSH private keys
Passwords
Tokens
Sensitive environment variables
Private infrastructure details
Unnecessary resource identifiers
```

The AWS material explicitly requires credentials to remain outside playbook source and Git repositories.

---

# 33. Validation Matrix

| Capability | Test | Expected Result | Evidence Level |
|---|---|---|---|
| Ansible installation | `ansible --version` | Ansible responds | Low |
| Connectivity | `ansible <host> -m ping` | Successful response | High |
| Group targeting | Group ping | Intended hosts respond | Medium |
| Syntax | `--syntax-check` | No syntax errors | Medium |
| Dry run | `-C` / `--check` | Simulation completes | Medium |
| Playbook execution | `ansible-playbook` | Successful play recap | High |
| Idempotency | Run twice | `changed` → `ok` | High |
| Variables | `debug` / `-vv` | Expected values resolved | Medium |
| Facts | `setup` | Expected host facts | Medium |
| Conditions | Multi-OS run | Correct tasks execute | High |
| Loops | Package loop | All items processed | Medium |
| Templates | Change variable | Configuration changes | High |
| Handlers | Change template | Handler executes | High |
| Role | Role execution | Expected state achieved | High |
| AWS dependency | Module execution | Dependencies available | Medium |
| AWS key pair | `ec2_key` | Key pair created | High |
| AWS EC2 | `ec2_instance` | Instance created | High |
| AWS convergence | Repeat run | No duplicate instance | High |
| Cleanup | AWS inspection | Temporary resources removed | Low |

---

# 34. Validation Boundaries

Successful validation establishes that the demonstrated Ansible automation works within the tested environment.

It does **not** establish:

```text
Production-scale reliability
        ✗

Enterprise Ansible Controller architecture
        ✗

Enterprise secrets management
        ✗

CI/CD integration
        ✗

Terraform infrastructure provisioning
        ✗

Kubernetes automation
        ✗

Multi-account AWS governance
        ✗

Production disaster recovery
        ✗

Production high availability
        ✗

Comprehensive automated testing
        ✗
```

The validation therefore supports the project's demonstrated capability without extrapolating to broader production claims.

---

# 35. Final Validation Checklist

Before considering the project sufficiently validated, confirm:

### Source and Structure

- [ ] Repository structure is complete.
- [ ] Playbooks are syntactically valid.
- [ ] Sensitive credentials and private keys are excluded.
- [ ] Environment-specific values are sanitized where necessary.

### Connectivity

- [ ] Control machine can reach intended Linux targets.
- [ ] SSH authentication works.
- [ ] Host/group targeting behaves correctly.

### Configuration Management

- [ ] Ad hoc modules execute successfully.
- [ ] Main playbooks execute successfully.
- [ ] Package state is correct.
- [ ] Service state is correct.
- [ ] File state is correct.

### Idempotency

- [ ] First execution produces expected changes.
- [ ] Repeated execution does not produce unnecessary changes.
- [ ] AWS `exact_count` prevents duplicate EC2 instances.

### Variables and Facts

- [ ] Variables resolve correctly.
- [ ] Debug output confirms important values.
- [ ] Facts are available.
- [ ] Conditions select the correct tasks.

### Loops

- [ ] Expected package/item collection is processed.
- [ ] Repeated execution remains idempotent.

### Templates and Handlers

- [ ] Template renders correctly.
- [ ] Configuration reaches the target.
- [ ] Configuration change triggers the handler.
- [ ] Unchanged configuration does not unnecessarily trigger the handler.

### Roles

- [ ] Role executes successfully.
- [ ] Role-local templates/files resolve correctly.
- [ ] Role preserves desired-state behavior.

### AWS

- [ ] Required AWS dependencies are installed.
- [ ] AWS authentication works.
- [ ] EC2 key pair automation works.
- [ ] Private key is handled without committing it.
- [ ] EC2 instance provisioning succeeds.
- [ ] Repeated execution does not create duplicate instances.
- [ ] Temporary AWS resources are cleaned up when no longer required.

### Evidence

- [ ] Personal execution evidence is separated from course material.
- [ ] Evidence supports major project claims.
- [ ] Screenshots are sanitized.
- [ ] Evidence collection is minimal and high-signal.

---

# 36. Final Validation Standard

The project should be considered validated when the evidence supports the following chain:

```text
Ansible Control Machine
        ↓
Inventory
        ↓
SSH Connectivity
        ↓
Target Selection
        ↓
Playbook Execution
        ↓
Desired State
        ↓
Idempotent Re-Execution
        ↓
Variables / Facts / Conditions
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
        ↓
AWS Resource Convergence
```

The strongest validation story is therefore not simply:

> **"The playbook ran successfully."**

It is:

> **"The automation could reach the intended targets, establish the desired state, adapt to runtime conditions, react to configuration changes, remain idempotent when repeated, execute through a reusable role structure, and extend the same desired-state approach to AWS resource provisioning."**

This reflects the validation mechanisms and practical progression established in the supplied Ansible learning material.
