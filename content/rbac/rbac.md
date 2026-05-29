## Structure

|Path                      |Purpose                                                                                                  |
|--------------------------|---------------------------------------------------------------------------------------------------------|
|[eda-ocp-sa](content/rbac/eda/eda-ocp-sa)/                    |OpenShift RBAC manifests for EDA — ClusterRole, ServiceAccount, ClusterRoleBinding, and token Secret             |
|[aac-ocp-sa](content/rbac/aac/aac-ocp-sa)/                    |OpenShift RBAC manifests for AAC — ClusterRole, ServiceAccount, ClusterRoleBinding, and token Secret             |
|[eda](content/rbac/eda/)/                    |OpenShift RBAC manifests for EDA and steps to EDA credentials |
|[acc](content/rbac/aac/)/                    |OpenShift RBAC manifests for AAC nd steps to AAC credential             |


## Overview

This directory contains the credential definitions needed by [EDA](content/rbac/eda) and [AAC](content/rbac/aac) to authenticate against the Target Openshift Cluster

Credentials in AAP are never exposed in plaintext to rulebooks — they are injected at runtime via file projection or environment variables, depending on the credential type.

-----
# EDA Credentials

|System                       |Why                                                    |Credential Type                    |Details                    |
|-----------------------------|-------------------------------------------------------|-----------------------------------|-----------------------------------
|OpenShift                    |To stream cluster events via the Juniper k8s.eda plugin|OpenShift Service Account Token  |[OpenShift Service Account Token](content/custom_credential_type/custom_credential_option_2.md)|
|Ansible Automation Controller|To fire job templates when a rulebook rule matches     |Red Hat Ansible Automation Platform|[aap credentials](content/rbac/eda/eda_aap_credentials.md) |

These are configured separately and both assigned to the same Rulebook Activation.

-----
# AAC  Credential

|System                       |Why                                                    |Credential Type                    |Details                    |
|-----------------------------|-------------------------------------------------------|-----------------------------------|-----------------------------------
|OpenShift                    |AAC OpenShift token is the identity credential that allows remediation playbooks to make changes in OpenShift on behalf of AAC|OpenShift/Kubernetes Bearer Token |[OCP Cluster token for AAC](content/rbac/aac/aac-ocp-credential.md)|


These are configured separately for the execution of the remediation of the playbooks of AAC Job templates.

## Further Reading

- [AAP EDA Credentials documentation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions/eda-credentials)
- [AAP Rulebook Activations](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions/eda-rulebook-activations)
- [Kubernetes Bearer Token credential type in AAP](https://docs.ansible.com/automation-controller/latest/html/userguide/credentials.html#openshift-or-kubernetes-api-bearer-token)
