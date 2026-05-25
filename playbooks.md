# [Playbooks](playbooks)

## Overview

The `playbooks/` directory contains the Ansible playbooks that form the **execution layer** of the event-driven governance platform. These playbooks are registered as **Job Templates** in Ansible Automation Controller (AAC) and are triggered by EDA rulebook rules — they never run manually in normal operation.

The separation is intentional:

- **EDA rulebooks** decide *when* to act (event matching and condition logic)
- **AAC playbooks** decide *how* to act (the actual remediation or notification steps)

This keeps the decision engine and execution engine cleanly decoupled. Playbooks can be tested, versioned, and audited independently of the rulebooks that invoke them.

-----

## Playbook Design Principles

All playbooks in this repository follow these conventions:

- **`hosts: localhost`** — playbooks run on the AAC execution node and interact with OpenShift via the `kubernetes.core` collection, not via SSH
- **`gather_facts: false`** — disabled to reduce startup time for fast remediation
- **Extra vars as inputs** — all context (namespace name, resource name, etc.) is passed in via `extra_vars` from the EDA rulebook action
- **Audit annotation** — every remediation writes a `eda.ansible.com/last-remediated` annotation back to the affected resource, creating an in-cluster audit trail
- **Idempotent** — playbooks can be run multiple times safely; they use `state: patched` or `state: present` rather than imperative operations

-----

## Playbook Structure

### Standard Playbook Template

```yaml
---
- name: <Descriptive remediation name>
  hosts: localhost
  gather_facts: false

  tasks:
    - name: <Primary remediation task>
      kubernetes.core.k8s:
        state: patched
        api_version: v1
        kind: <ResourceKind>
        name: "{{ resource_name }}"
        namespace: "{{ namespace }}"
        definition:
          metadata:
            <labels or annotations to apply>

    - name: Write audit annotation
      kubernetes.core.k8s:
        state: patched
        api_version: v1
        kind: <ResourceKind>
        name: "{{ resource_name }}"
        namespace: "{{ namespace }}"
        definition:
          metadata:
            annotations:
              eda.ansible.com/last-remediated: "{{ ansible_date_time.iso8601 }}"
              eda.ansible.com/remediation-reason: "<human-readable reason>"
```

-----

## Playbook Reference

### Playbook: [apply_enterprise_compliance_Quotas.yml](playbooksapply_enterprise_compliance_Quotas.yml)

-----

## Registering Playbooks as AAC Job Templates

For each playbook, create a corresponding Job Template in AAC:

1. Navigate to **Automation Execution → Templates → Create job template**
1. Configure as follows:
   
   |Field                |Value                                                                   |
   |---------------------|------------------------------------------------------------------------|
   |Name                 |Match exactly what the EDA rulebook specifies in `run_job_template.name`|
   |Job Type             |`Run`                                                                   |
   |Inventory            |`localhost` inventory (or an inventory with your OpenShift hosts)       |
   |Project              |Your project pointing to this repository                                |
   |Playbook             |Path to the playbook file (e.g. `playbooks/restore-network-policy.yml`) |
   |Execution Environment|An EE with `kubernetes.core` collection installed                       |
   |Extra Variables      |Check **Prompt on launch** — EDA passes extra vars at job launch time   |
   |Credentials          |OpenShift credential (for `kubernetes.core.k8s` authentication)         |
1. Click **Save**

> **⚠️ Important**
> The Job Template name in AAC must **exactly match** the name specified in the EDA rulebook’s `run_job_template.name` field. A mismatch causes the rule action to fail silently or with a 404 error.

-----

## Execution Environment Requirements

The playbooks use the `kubernetes.core` collection. Your AAC Execution Environment must include:

```yaml
# In your EE definition (execution-environment.yml)
dependencies:
  galaxy:
    collections:
      - name: kubernetes.core
  python:
    - kubernetes
    - openshift
```

The `kubernetes.core` collection requires the `kubernetes` and `openshift` Python packages to be present in the EE.

-----

## Audit Trail

Every remediation playbook writes back to the affected resource in OpenShift:

```yaml
metadata:
  annotations:
    eda.ansible.com/last-remediated: "2025-06-01T14:32:11Z"
    eda.ansible.com/remediation-reason: "NetworkPolicy default-deny was deleted and restored from Git"
```

This creates a **dual audit trail**:

- **In-cluster** — annotations on every remediated resource, visible in `oc describe` or the OpenShift Web Console
- **In AAC** — every job run is logged with full output, timing, triggered-by information, and extra vars (minus secrets)

-----

## Further Reading

- [kubernetes.core collection](https://docs.ansible.com/ansible/latest/collections/kubernetes/core/index.html)
- [AAP Job Templates](https://docs.ansible.com/automation-controller/latest/html/userguide/job_templates.html)
- [AAP Execution Environments](https://ansible.readthedocs.io/projects/builder/en/latest/)
- [Ansible idempotency guide](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-Idempotency)
