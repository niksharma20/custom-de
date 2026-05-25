# Playbooks

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

### Playbook: Remediate PVC Labels

**Job Template name in AAC:** `NaaS - Remediate PVC Labels`

Triggered when a PersistentVolumeClaim is created in a governed namespace without the required `storage-class-approved` label.

**Extra vars received from EDA:**

|Variable   |Description                        |
|-----------|-----------------------------------|
|`namespace`|Namespace where the PVC was created|
|`pvc_name` |Name of the PVC                    |

**What it does:**

1. Patches the PVC to add the `storage-class-approved: "true"` label
1. Writes a `eda.ansible.com/last-remediated` audit annotation

-----

### Playbook: Remediate TLS Certificate Annotation

**Job Template name in AAC:** `NaaS - Remediate TLS Certificate Annotation`

Triggered when a `kubernetes.io/tls` Secret is created without the `cert-manager.io/certificate-name` annotation.

**Extra vars received from EDA:**

|Variable     |Description                           |
|-------------|--------------------------------------|
|`namespace`  |Namespace where the Secret was created|
|`secret_name`|Name of the TLS Secret                |

**What it does:**

1. Reads the existing Secret to confirm it is TLS type
1. Derives the expected certificate name by stripping `-tls` from the secret name
1. Patches the Secret with the `cert-manager.io/certificate-name` annotation
1. Writes audit annotations

-----

### Playbook: Restore NetworkPolicy

**Job Template name in AAC:** `NaaS - Restore NetworkPolicy`

Triggered when a NetworkPolicy is deleted from a governed namespace.

**Extra vars received from EDA:**

|Variable     |Description                                  |
|-------------|---------------------------------------------|
|`namespace`  |Namespace where the NetworkPolicy was deleted|
|`policy_name`|Name of the deleted NetworkPolicy            |

**What it does:**

1. Fetches the NetworkPolicy YAML from the GitOps repository (Git is the source of truth)
1. Re-applies the policy to the namespace using `kubernetes.core.k8s`
1. Annotates the namespace with the remediation event and reason

> **💡 Tip**
> This playbook embodies the GitOps principle even during automated remediation — the policy definition comes from Git, not from memory or local state. If the Git copy is updated, the next remediation will apply the updated version automatically.

-----

### Playbook: Namespace Onboarding Notification

**Job Template name in AAC:** `NaaS - Namespace Onboarding Notification`

Triggered when a new namespace with `eda-governed: "true"` is detected (i.e. a namespace just provisioned via the self-service workflow with EDA governance enabled).

**Extra vars received from EDA:**

|Variable   |Description                                |
|-----------|-------------------------------------------|
|`namespace`|Name of the new namespace                  |
|`owner`    |Owner/requester (from namespace annotation)|

**What it does:**

1. Sends a notification (e.g. Slack, email, or ITSM ticket) confirming the namespace is now EDA-governed
1. Optionally creates an initial audit log entry in your ITSM system

-----

### Playbook: Flag Insecure Route

**Job Template name in AAC:** `NaaS - Flag Insecure Route`

Triggered when a Route is created without TLS configuration in a governed namespace.

**Extra vars received from EDA:**

|Variable    |Description                          |
|------------|-------------------------------------|
|`namespace` |Namespace where the Route was created|
|`route_name`|Name of the insecure Route           |

**What it does:**

1. Annotates the Route with a `security.governance/insecure-route: "flagged"` label
1. Sends a notification to the namespace owner
1. Optionally creates a remediation ticket in the issue tracker

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