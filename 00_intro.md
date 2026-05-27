# Custom Decision Environment for Event-Driven Ansible on OpenShift

## Introduction

This repository contains everything needed to build, configure, and operate a **Custom Decision Environment (DE)** for Ansible Event-Driven Automation (EDA) for OpenShift.

A Decision Environment is a container image that packages the Ansible EDA runtime, event source plugins, Python dependencies, and system libraries needed to execute rulebooks in the Ansible Automation Platform (AAP) Decision Controller. This repository provides a **custom DE** that extends the Red Hat supported base image with the [Juniper k8s.eda](https://github.com/Juniper/k8s.eda) event source plugin — enabling real-time Kubernetes and OpenShift resource event streaming directly into EDA rulebooks.

This is the **infrastructure backbone** for the EDA governance layer described in the Namespace as a Service platform. When a namespace is labelled `eda-governed=true` at provisioning time, it is this Decision Environment and its rulebooks that watch the cluster and trigger remediation automatically.

-----

## What This Repository Does

```
Custom Decision Environment (container image)
          ↓
Packages: juniper.eda.k8s + ansible.eda + kubernetes_asyncio
          ↓
Deployed to: AAP Automation Decisions (EDA Decision Controller)
          ↓
Authenticates to OpenShift via: ServiceAccount + ClusterRole + Bearer Token
          ↓
Watches: Namespaces, Routes, PVCs, Secrets, NetworkPolicies (labelled resources)
          ↓
Fires rulebook rules → triggers AAC Job Templates → remediates cluster state
```

-----

## Repository Structure

|Path                      |Purpose                                                                                                  |
|--------------------------|---------------------------------------------------------------------------------------------------------|
|`decision-environment.yml`|The DE build manifest — defines the base image, collections, Python deps, and system packages            |
|`eda_credentials/`        |AAP credential type definitions for EDA — structured YAML for importing into AAP                         |
|`eda_custom_credential/`  |Custom credential type definitions — extends AAP with OpenShift token and EDA-specific credential schemas|
|`ocp/`                    |OpenShift RBAC manifests — ClusterRole, ServiceAccount, ClusterRoleBinding, and token Secret             |
|`rulebooks/`              |EDA rulebooks — event source configuration and rules that map cluster events to AAC job templates        |
|`playbooks/`              |Ansible playbooks — remediation logic executed by AAC when a rulebook rule fires                         |
|`extra/`                  |Supporting files — additional configuration, utilities, or reference material                            |
|`00_ocp_execution.md`     |Step-by-step OpenShift setup guide — apply RBAC, generate tokens, label namespaces                       |

-----

## Goals and Objectives

### Goals

1. **Enable real-time cluster governance** — move from periodic polling to event-driven reactions to cluster state changes using a custom EDA Decision Environment with the Juniper k8s event source.
1. **Provide a reusable, version-controlled DE** — the `decision-environment.yml` manifest is the single source of truth for the DE image. Any change to collections, Python deps, or the base image is tracked in Git and reproducible.
1. **Establish least-privilege OpenShift access** — the RBAC manifests in `ocp/` give the EDA controller the minimum permissions needed to watch cluster resources — read-only `get`, `list`, `watch` — with no write access.
1. **Bridge EDA and AAC cleanly** — rulebooks in this repo do not contain remediation logic themselves; they fire AAC job templates. This keeps decision logic (EDA) and execution logic (AAC + playbooks) cleanly separated.
1. **Support the Namespace as a Service governance model** — when namespaces are provisioned with `eda-governed=true`, this DE and its rulebooks automatically activate governance without any additional manual configuration.

### Objectives

By working through this repository, you will be able to:

- Build and publish a custom EDA Decision Environment image using `ansible-builder`
- Register the DE in AAP Automation Decisions
- Configure the required OpenShift RBAC objects so EDA can authenticate and watch resources
- Import and activate EDA credential types for OpenShift token-based authentication
- Activate a rulebook in AAP that streams live events from OpenShift
- Understand how the `kubeconfig` credential projection works in AAP
- Trigger and verify remediation playbooks via AAC job templates

-----

## Prerequisites

Before working through this repository, ensure you have:

- Access to an **OpenShift cluster** with cluster-admin or equivalent permissions
- **Ansible Automation Platform 2.4+** with both Automation Controller (AAC) and Automation Decisions (EDA) deployed
- The `oc` CLI authenticated to your OpenShift cluster
- The `ansible-builder` CLI installed (for building the DE image)
- A container registry accessible from your AAP instance (e.g. Quay, OpenShift internal registry)

-----

## Quick Start Sequence

```
1. Build the Decision Environment     → decision-environment.yml
2. Push image to your registry        → ansible-builder build + podman push
3. Register DE in AAP                 → Automation Decisions → Decision Environments
4. Apply OpenShift RBAC               → ocp/ manifests
5. Extract ServiceAccount token       → 00_ocp_execution.md Step 5
6. Configure EDA credentials in AAP  → eda_credentials/ + eda_custom_credential/
7. Create AAC Job Templates           → playbooks/
8. Import rulebook project in AAP    → rulebooks/
9. Activate rulebook                  → Automation Decisions → Rulebook Activations
10. Label target namespaces           → oc label namespace <name> type=eda
```

-----

## Further Reading

- [Ansible EDA documentation](https://www.ansible.com/products/event-driven-ansible)
- [ansible-builder documentation](https://ansible.readthedocs.io/projects/builder/en/latest/)
- [Juniper k8s.eda collection](https://github.com/Juniper/k8s.eda)
- [kubernetes_asyncio Python library](https://github.com/tomplus/kubernetes_asyncio)
- [AAP Decision Environments](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions)
- [OpenShift RBAC documentation](https://docs.openshift.com/container-platform/latest/authentication/using-rbac.html)
