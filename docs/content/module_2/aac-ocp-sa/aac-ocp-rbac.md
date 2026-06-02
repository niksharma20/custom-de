# OpenShift RBAC Setup for AAC

## Overview

For the EDA Decision Controller to stream events from an OpenShift cluster, it needs a **machine identity** (ServiceAccount) with read-only permissions to watch cluster resources, and a **persistent bearer token** it can use to authenticate.

The `ocp/` directory contains the four Kubernetes manifests that establish this access, and `00_ocp_execution.md` documents the exact sequence of commands to apply them.

This page covers both: the manifests and the execution steps.

-----

## Why This Setup is Needed

OpenShift does not automatically issue long-lived tokens to external applications. By default, ServiceAccount tokens are short-lived and rotated. Since the EDA Decision Controller runs **outside** the cluster (in AAP), it needs a manually created, persistent Secret-backed token to maintain a stable connection to the OpenShift API.

Additionally, the Juniper `k8s.eda` event source plugin uses label selectors to filter which resources it watches. Unlabelled resources are ignored — this is by design to limit event noise and reduce API server load.

-----

## Step 1: Create the ClusterRole

**File:** [01_aac-clusterrole.yaml](content/rbac/aac/aac-ocp-sa/01_aac-clusterrole.yaml)

The ClusterRole defines exactly what the AAC service account is allowed to do — **read-only access** to the resource types the rulebooks need to watch.

**Apply:**

```bash
oc apply -f rbac/aac/aac-ocp-sa/01_aac-clusterrole.yaml
```

> **ℹ️ Note**
> Only include resource types in the ClusterRole that your rulebooks actively watch. Granting unnecessary permissions widens the blast radius if the token is ever compromised.

-----

## Step 2: Create the ServiceAccount

**File:** [02_aac-serviceaccount.yaml](content/rbac/aac/aac-ocp-sa/02_aac-serviceaccount.yaml)

The ServiceAccount is the machine identity the EDA controller presents when connecting to OpenShift. It is created in a dedicated namespace (e.g. `aap`) to keep it isolated from workload namespaces.  

**Apply:**

```bash
# Create the namespace first if it doesn't exist
oc create namespace aap

# Create the ServiceAccount
oc apply -f rbac/aac/aac-ocp-sa/02_aac-serviceaccount.yaml
```

-----

## Step 3: Bind the ServiceAccount to the ClusterRole

**File:** [03_aac-rolebinding.yaml](content/rbac/aac/aac-ocp-sa/03_aac-rolebinding.yaml)

The ClusterRoleBinding connects the ServiceAccount (Step 2) to the ClusterRole (Step 1), granting the identity its read-only permissions.

**Apply:**

```bash
oc apply -f rbac/aac/aac-ocp-sa/03_aac-rolebinding.yaml
```

-----

## Step 4: Generate a Persistent Long-Lived Token

**File:** [04_aac-token-secret.yaml](content/rbac/aac/aac-ocp-sa/04_aac-token-secret.yaml)

OpenShift does not auto-generate permanent tokens for ServiceAccounts. Since the EDA controller lives outside the cluster, you must manually create a Secret to house a permanent bearer token.

**Apply:**

```bash
oc apply -f rbac/aac/aac-ocp-sa/04_aac-token-secret.yaml
```

> **⚠️ Important**
> This token does not expire automatically. Treat it like a password — store it only in AAP as a credential, rotate it periodically, and never commit the decoded value to Git.

-----

## Step 5: Extract the Token

Once the Secret is created, extract and decode the token for use in AAP:

```bash
oc get secret aac-token-secret -n aap \
  -o jsonpath='{.data.token}' | base64 --decode
```

Copy the output. You will paste it into AAP when configuring the OpenShift credential in the next section.

You can also retrieve the cluster API URL at this point:

```bash
oc whoami --show-server
```

-----
-----


## Further Reading

- [OpenShift RBAC documentation](https://docs.openshift.com/container-platform/latest/authentication/using-rbac.html)
- [Kubernetes ServiceAccount tokens](https://kubernetes.io/docs/concepts/configuration/secret/#service-account-token-secrets)
- [AAP Credential Types — OpenShift/Kubernetes Bearer Token](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions/eda-credentials)
