# Custom Decision Environment for Event-Driven Ansible on OpenShift

> *Building self-governing namespaces with GitOps, Orchestrator, and Event-Driven Ansible — the EDA governance layer of a Namespace as a Service platform.*

-----

## What This Repository Is

This repository contains everything needed to build, configure, and operate a **Custom Decision Environment (DE)** for Ansible Event-Driven Automation (EDA) on OpenShift Container Platform.

A Decision Environment is a container image that packages the EDA runtime, event source plugins, Python dependencies, and system libraries required to execute rulebooks in the Ansible Automation Platform (AAP) Decision Controller.

This custom DE extends the Red Hat supported base image with the [Juniper k8s.eda](https://github.com/Juniper/k8s.eda) event source plugin — enabling real-time Kubernetes and OpenShift resource event streaming directly into EDA rulebooks.

When a namespace is labelled `eda-governed=true` at provisioning time, this Decision Environment and its rulebooks automatically watch the cluster and trigger remediation via Ansible Automation Controller (AAC) — no human intervention required.

-----

## The Governance Loop

```
OpenShift Events (eda-governed=true namespaces)
              ↓
   Event-Driven Ansible (this DE)
   juniper.eda.k8s event source
              ↓  Trigger Action
   Automation Controller (AAC)
   Remediation Playbooks
              ↓  Ansible Automation
   OpenShift Container Platform
```

-----

## Repository Structure

|Path                                                    |Purpose                                                                                   |
|--------------------------------------------------------|------------------------------------------------------------------------------------------|
|[`decision-environment.yml`](./decision-environment.yml)|DE build manifest — base image, collections, Python deps, system packages                 |
|[`ocp/`](./ocp/)                                        |OpenShift RBAC — ClusterRole, ServiceAccount, ClusterRoleBinding, token Secret            |
|[`rulebooks/`](./rulebooks/)                            |EDA rulebooks — event source config and rules that map cluster events to AAC job templates|
|[`playbooks/`](./playbooks/)                            |Ansible remediation playbooks — executed by AAC when a rulebook rule fires                |
|[`eda_credentials/`](./eda_credentials/)                |AAP credential definitions for OpenShift and AAC                                          |
|[`eda_custom_credential/`](./eda_custom_credential/)    |Custom credential type schemas — extends AAP with org-specific credential fields          |
|[`extra/`](./extra/)                                    |Supporting config, utilities, and reference material                                      |
|[`00_ocp_execution.md`](./00_ocp_execution.md)          |Step-by-step OpenShift setup — RBAC, token generation, namespace labelling                |

-----

## Prerequisites

Before working through this repository, ensure you have:

- Access to an **OpenShift Container Platform** cluster with cluster-admin permissions
- **Ansible Automation Platform 2.4+** with both Automation Controller (AAC) and Automation Decisions (EDA) deployed
- The `oc` CLI authenticated to your OpenShift cluster
- The `ansible-builder` CLI installed
- A container registry accessible from your AAP instance (e.g. Quay, OpenShift internal registry)

-----

## Quick Start

```
1. Build the Decision Environment     →  decision-environment.yml
2. Push image to your registry        →  ansible-builder build + podman push
3. Register DE in AAP                 →  Automation Decisions → Decision Environments
4. Apply OpenShift RBAC               →  ocp/ manifests
5. Extract ServiceAccount token       →  00_ocp_execution.md
6. Configure EDA credentials in AAP  →  eda_credentials/ + eda_custom_credential/
7. Create AAC Job Templates           →  playbooks/
8. Import rulebook project in AAP    →  rulebooks/
9. Activate rulebook                  →  Automation Decisions → Rulebook Activations
10. Label target namespaces           →  oc label namespace <name> eda-governed=true
```

-----

## AAP Setup Checklist

### Automation Controller (AAC)

> [!NOTE]
> Configure `ansible.cfg` before starting. See the [Getting Started with Ansible Content Collections](https://developers.redhat.com/learning/learn:ansible:getting-started-ansible-content-collections/resource/resources:finding-and-installing-collections-and-using-them-playbooks?source=sso) course for guidance.

- [ ] Organisation created (e.g. `Application Development`)
- [ ] Credentials configured (OpenShift token, SCM)
- [ ] Project created and synced (pointing to this repo)
- [ ] Job Templates created — one per remediation playbook in `playbooks/`
- [ ] Inventory configured (`localhost` for `kubernetes.core` playbooks)
- [ ] Execution Environment with `kubernetes.core` assigned globally or per template

### Automation Decisions (EDA)

- [ ] Credential Type — `OpenShift Token (EDA)` created from `eda_custom_credential/`
- [ ] Credential — OpenShift Bearer Token configured
- [ ] Credential — Red Hat Ansible Automation Platform (AAC connection) configured
- [ ] Decision Environment registered (image built from `decision-environment.yml`)
- [ ] Project created and synced (pointing to `rulebooks/`)
- [ ] Rulebook Activation created with both credentials and the custom DE assigned

-----

## Important Notes

> [!IMPORTANT]
> The `aap` namespace must exist in OpenShift before applying the ServiceAccount manifest. The `kubernetes_asyncio` Python package is critical — without it the Juniper plugin cannot open an event stream and the rulebook activation will fail silently at startup.

> [!WARNING]
> Never commit decoded tokens, bearer credentials, or cluster URLs to this repository. Use AAP credential objects and the `{{ eda.filename.kubeconfig }}` injection pattern — credentials are never hardcoded in rulebook YAML. Rotate the ServiceAccount token periodically.

> [!TIP]
> The AAC Job Template name must **exactly** match the `run_job_template.name` value in the rulebook — including capitalisation and spaces. A mismatch causes the rule action to fail with a 404 error against the AAC API.

-----

## Documentation

|Document                                                                                    |Description                                        |
|--------------------------------------------------------------------------------------------|---------------------------------------------------|
|[`00_ocp_execution.md`](./00_ocp_execution.md)                                              |Full OpenShift setup walkthrough                   |
|[`AI Collaboration Diligence Statement.md`](./AI%20Collaboration%20Diligence%20Statement.md)|Transparency statement on AI-assisted documentation|
|[`LICENSE`](./LICENSE)                                                                      |Apache 2.0                                         |

-----

## Further Reading

- [Ansible EDA documentation](https://www.ansible.com/products/event-driven-ansible)
- [ansible-builder documentation](https://ansible.readthedocs.io/projects/builder/en/latest/)
- [Juniper k8s.eda collection](https://github.com/Juniper/k8s.eda)
- [kubernetes_asyncio](https://github.com/tomplus/kubernetes_asyncio)
- [AAP Automation Decisions](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions)
- [OpenShift RBAC documentation](https://docs.openshift.com/container-platform/latest/authentication/using-rbac.html)

-----

> *Documentation in this repository was produced with AI assistance. See the [AI Collaboration Diligence Statement](./AI%20Collaboration%20Diligence%20Statement.md) for full details.*