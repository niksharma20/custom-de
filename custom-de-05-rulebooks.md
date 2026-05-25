# Rulebooks

## Overview

The `rulebooks/` directory contains the EDA rulebook definitions that form the **decision layer** of the event-driven governance platform. Each rulebook defines:

- **Sources** — where events come from (the Juniper `k8s.eda` plugin watching OpenShift)
- **Rules** — conditions that match incoming events
- **Actions** — what to do when a rule matches (fire an AAC job template)

Rulebooks are imported into AAP as a **Project** and activated via **Rulebook Activations** in Automation Decisions.

-----

## Key Rulebook: `openshifteda.yml`

This is the primary rulebook in the repository. It connects to OpenShift using the Juniper `k8s.eda` event source and fires AAC job templates in response to cluster state changes on labelled resources.

### The Ingestion Layer (Sources)

```yaml
sources:
  - name: Listen for OpenShift events
    juniper.eda.k8s:
      kubeconfig: "{{ eda.filename.kubeconfig }}"
      kinds:
        - api_version: v1
          kind: Namespace
        - api_version: v1
          kind: Route
      label_selectors:
        - "type=eda"
```

**What this does:**

When the rulebook activates, the `juniper.eda.k8s` source plugin:

1. Reads the kubeconfig file injected by AAP from `{{ eda.filename.kubeconfig }}`
1. Presents the ServiceAccount bearer token to the OpenShift API
1. Opens an async HTTP stream (WebSocket) to the OpenShift API server using `kubernetes_asyncio`
1. Begins receiving live lifecycle events (`ADDED`, `MODIFIED`, `DELETED`) for the specified resource types
1. Filters events to only those on resources labelled `type=eda`

This is **not polling** — OpenShift pushes events to the stream in real time as they occur.

### Event Structure

Each event received from the stream has the following structure, accessible in rule conditions via `event.*`:  
Below is an example.  

```yaml
event:
  type: "ADDED"           # ADDED | MODIFIED | DELETED
  resource:
    kind: "Namespace"
    metadata:
      name: "team-payments-dev"
      namespace: "team-payments-dev"
      labels:
        type: "eda"
      annotations:
        eda.ansible.com/watch-events: "PersistentVolumeClaim,Secret,NetworkPolicy"
    spec: { ... }
    status: { ... }
```

### Rules

Rules follow the structure:

```yaml
rules:
  - name: <descriptive rule name>
    condition: <jinja2-style boolean expression on event.*>
    action:
      run_job_template:
        name: "<AAC job template name>"
        organization: "<AAC organisation>"
        job_args:
          extra_vars:
            <key>: "{{ event.resource.metadata.<field> }}"
```

#### Example Rule: New Namespace Created

```yaml
- name: New governed namespace detected
  condition: >
    event.type == "ADDED" and
    event.resource.kind == "Namespace" and
    event.resource.metadata.labels["eda-governed"] == "true"
  action:
    run_job_template:
      name: "NaaS - Namespace Onboarding Notification"
      organization: Platform
      job_args:
        extra_vars:
          namespace: "{{ event.resource.metadata.name }}"
          owner: "{{ event.resource.metadata.annotations['provisioned-by'] | default('unknown') }}"
```

#### Example Rule: Route Created Without TLS

```yaml
- name: Route created without TLS
  condition: >
    event.type == "ADDED" and
    event.resource.kind == "Route" and
    event.resource.spec.tls is not defined
  action:
    run_job_template:
      name: "NaaS - Flag Insecure Route"
      organization: Platform
      job_args:
        extra_vars:
          namespace: "{{ event.resource.metadata.namespace }}"
          route_name: "{{ event.resource.metadata.name }}"
```

-----

## How the Three Layers Map Together

```
OpenShift API (event stream)
        ↓
juniper.eda.k8s source plugin
        ↓  (filters by label: type=eda)
Rule condition evaluated
        ↓  (match)
run_job_template action
        ↓
AAC authenticates via AAP credential
        ↓
AAC Job Template runs playbook
        ↓
Cluster state remediated
```

-----

## Rulebook Variable Reference

|Variable                                   |Source                  |Description                                                            |
|-------------------------------------------|------------------------|-----------------------------------------------------------------------|
|`{{ eda.filename.kubeconfig }}`            |AAP credential injection|Path to the kubeconfig file written by AAP at activation time          |
|`{{ event.type }}`                         |Juniper k8s source      |Event type: `ADDED`, `MODIFIED`, or `DELETED`                          |
|`{{ event.resource.kind }}`                |Juniper k8s source      |Kubernetes resource type (e.g. `Namespace`, `Route`, `Pod`)            |
|`{{ event.resource.metadata.name }}`       |Juniper k8s source      |Name of the resource that triggered the event                          |
|`{{ event.resource.metadata.namespace }}`  |Juniper k8s source      |Namespace the resource belongs to                                      |
|`{{ event.resource.metadata.labels }}`     |Juniper k8s source      |Labels on the resource — used in conditions and passed to job templates|
|`{{ event.resource.metadata.annotations }}`|Juniper k8s source      |Annotations on the resource                                            |
|`{{ event.resource.spec }}`                |Juniper k8s source      |Full spec of the resource (varies by kind)                             |

-----

## Importing Rulebooks into AAP

### Step 1: Create a Project in Automation Decisions

1. Navigate to **Automation Decisions → Projects → Create project**
1. Set **Source control type** to `Git`
1. Set **Source control URL** to your fork of this repository (or the URL where your rulebooks are hosted)
1. Set **Source control branch** to `main`
1. If the repo is private, assign an **SCM credential**
1. Click **Save** — AAP will sync the project and make the rulebooks available

### Step 2: Create a Rulebook Activation

1. Navigate to **Automation Decisions → Rulebook Activations → Create rulebook activation**
1. Select your **Project**
1. Select the **Rulebook** (e.g. `openshifteda.yml`)
1. Select your **Decision Environment** (the custom DE image)
1. Assign **Credentials**:
- OpenShift Bearer Token credential
- Red Hat Ansible Automation Platform credential
1. Set **Restart policy** to `Always`
1. Click **Create rulebook activation**

The activation will start immediately. Check the **History** tab to confirm it connected to OpenShift successfully and is streaming events.

-----

## Troubleshooting

|Symptom                              |Likely Cause                               |Resolution                                                                |
|-------------------------------------|-------------------------------------------|--------------------------------------------------------------------------|
|Activation fails to start            |Missing or invalid OpenShift credential    |Re-check token and cluster URL in credential                              |
|No events received                   |No resources labelled `type=eda`           |Apply `oc label namespace <name> type=eda`                                |
|`kubernetes_asyncio` import error    |DE missing Python dependency               |Rebuild the DE with `kubernetes_asyncio` in `decision-environment.yml`    |
|Rule fires but job template not found|AAC job template name mismatch             |Ensure template name in rulebook exactly matches name in AAC              |
|Rule fires but AAC auth fails        |AAC credential missing or wrong permissions|Check AAC credential and ensure the user has `Execute` on the job template|

-----

## Further Reading

- [Juniper k8s.eda collection](https://github.com/Juniper/k8s.eda)
- [AAP Rulebook Activations](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions/eda-rulebook-activations)
- [Serverless Workflow Condition expressions](https://ansible.readthedocs.io/projects/rulebook/en/latest/conditions.html)
- [EDA `run_job_template` action](https://ansible.readthedocs.io/projects/rulebook/en/latest/actions.html#run-job-template)
