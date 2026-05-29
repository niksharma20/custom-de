## Overview

The `rbac` directory contains the credential definitions needed by [EDA](content/rbac/eda) and [AAC](content/rbac/aac) to authenticate against the Target Openshift Cluster

Credentials in AAP are never exposed in plaintext to rulebooks — they are injected at runtime via file projection or environment variables, depending on the credential type.

-----

## Why Two Credential Types Are Needed

|System                       |Why                                                    |Credential Type                    |
|-----------------------------|-------------------------------------------------------|-----------------------------------|
|OpenShift                    |To stream cluster events via the Juniper k8s.eda plugin|OpenShift Service Account Token  |
|Ansible Automation Controller|To fire job templates when a rulebook rule matches     |Red Hat Ansible Automation Platform|

These are configured separately and both assigned to the same Rulebook Activation.

-----

## Credential 1: [OpenShift Service Account Token](content/custom_credential_type/custom_credential_option_2.md)

This credential gives the EDA Decision Controller its identity when connecting to the OpenShift API.

## Credential 2: [Red Hat Ansible Automation Platform](content/eda_credentials/aap-credential.md)  

### What to configure in AAP

Navigate to **Automation Decisions → Credentials → Create credential** and select **OpenShift or Kubernetes API Bearer Token** as the credential type.

|Field                               |Value                                                  |
|------------------------------------|-------------------------------------------------------|
|Name                                |`OpenShift EDA Watcher` (or your preferred name)       |
|Credential Type                     |`OpenShift or Kubernetes API Bearer Token`             |
|OpenShift or Kubernetes API Endpoint|Your cluster API URL (from `oc whoami --show-server`)  |
|API authentication bearer token     |The decoded token extracted in OCP RBAC Step 5         |
|Verify SSL                          |Enabled (recommended) or disabled for self-signed certs|

### How it is used in the rulebook

When this credential is assigned to a Rulebook Activation, AAP:

1. Reads the bearer token and API endpoint from the credential
1. Dynamically writes a `kubeconfig` file into the container runtime
1. Exposes the file path as `{{ eda.filename.kubeconfig }}`

The rulebook references it like this:

```yaml
sources:
  - juniper.eda.k8s:
      kubeconfig: "{{ eda.filename.kubeconfig }}"
      kinds:
        - api_version: v1
          kind: Namespace
```

> **ℹ️ Note**
> You never hardcode the token or cluster URL in the rulebook YAML. AAP’s credential projection handles this at activation time — keeping secrets out of source control entirely.

-----

## Credential 2: Red Hat Ansible Automation Platform (AAC Token)

This credential allows the EDA rulebook to fire job templates in Ansible Automation Controller when a rule condition is matched.

### What to configure in AAP

Navigate to **Automation Decisions → Credentials → Create credential** and select **Red Hat Ansible Automation Platform** as the credential type.

|Field                              |Value                                            |
|-----------------------------------|-------------------------------------------------|
|Name                               |`AAC EDA Integration` (or your preferred name)   |
|Credential Type                    |`Red Hat Ansible Automation Platform`            |
|Red Hat Ansible Automation Platform|URL of your AAC instance                         |
|Username                           |AAC user with permissions to launch job templates|
|Password                           |Password for the above user                      |


> **💡 Tip**
> In production, create a dedicated service account in AAC with the minimum permissions required — specifically `Execute` permission on the job templates the rulebooks need to trigger. Avoid using an admin account.

### How it is used in the rulebook

When a rule fires, the `run_job_template` action uses this credential to authenticate against AAC:

```yaml
  rules:
    - name: Set Resource Quotas to a Namespace
      condition: >
        event.resource.kind == "Namespace" and (event.type == "ADDED" or event.type == "MODIFIED") and
        event.resource.metadata.labels.type == "eda"
      action:
        run_job_template:
          name: OpenShift Set Resource Quota on Namespace
          organization: "Default"
          job_args:
            extra_vars:
              namespace: "{{ event.resource.metadata.name }}"
```

AAP reads the AAC credential assigned to the activation and injects the connection details automatically — no URL or token appears in the rulebook YAML.

-----

## Assigning Credentials to a Rulebook Activation

When creating a Rulebook Activation in AAP:

1. Navigate to **Automation Decisions → Rulebook Activations → Create rulebook activation**
1. Select your **Project** and **Rulebook**
1. Select your **Decision Environment** (the custom DE built from `decision-environment.yml`)
1. Under **Credentials**, assign **both** credentials:
- `OpenShift EDA Watcher` (OpenShift Bearer Token)
- `AAC EDA Integration` (Red Hat Ansible Automation Platform)
1. Set **Restart policy** to `Always` for production activations

> **⚠️ Warning**
> If either credential is missing or misconfigured, the Rulebook Activation will fail at startup. Check the activation log in **Automation Decisions → Rulebook Activations → [your activation] → History** for authentication errors.

-----

## Credential Summary

```
Rulebook Activation
      ├── Decision Environment  →  custom-eda-de:latest
      ├── OpenShift Credential  →  kubeconfig injected as {{ eda.filename.kubeconfig }}
      └── AAC Credential        →  run_job_template authenticates to AAC automatically
```

-----

## Further Reading

- [AAP EDA Credentials documentation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions/eda-credentials)
- [AAP Rulebook Activations](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions/eda-rulebook-activations)
- [Kubernetes Bearer Token credential type in AAP](https://docs.ansible.com/automation-controller/latest/html/userguide/credentials.html#openshift-or-kubernetes-api-bearer-token)
