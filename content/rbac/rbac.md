## Overview

The `rbac` directory contains the credential definitions needed by [EDA](content/rbac/eda) and [AAC](content/rbac/aac) to authenticate against the Target Openshift Cluster

Credentials in AAP are never exposed in plaintext to rulebooks — they are injected at runtime via file projection or environment variables, depending on the credential type.

-----
# EDA  

## Two Credential

|System                       |Why                                                    |Credential Type                    |
|-----------------------------|-------------------------------------------------------|-----------------------------------|
|OpenShift                    |To stream cluster events via the Juniper k8s.eda plugin|OpenShift Service Account Token  |
|Ansible Automation Controller|To fire job templates when a rulebook rule matches     |Red Hat Ansible Automation Platform|

These are configured separately and both assigned to the same Rulebook Activation.

-----

## Credential 1: [OpenShift Service Account Token](content/custom_credential_type/custom_credential_option_2.md)

This credential gives the EDA Decision Controller its identity when connecting to the OpenShift API.

## Credential 2: [aap credentials](content/rbac/eda/eda_aap_credentials.md)  

-----

-----
# ACC  

## ..

## Further Reading

- [AAP EDA Credentials documentation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions/eda-credentials)
- [AAP Rulebook Activations](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions/eda-rulebook-activations)
- [Kubernetes Bearer Token credential type in AAP](https://docs.ansible.com/automation-controller/latest/html/userguide/credentials.html#openshift-or-kubernetes-api-bearer-token)
