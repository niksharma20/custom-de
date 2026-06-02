# OpenShift RBAC Setup for EDA

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

**File:** [ocp/01_eda-clusterrole.yaml](ocp/01_eda-clusterrole.yaml)

The ClusterRole defines exactly what the EDA service account is allowed to do — **read-only access** to the resource types the rulebooks need to watch.

**Apply:**

```bash
oc apply -f ocp/01_eda-clusterrole.yaml
```

> **ℹ️ Note**
> Only include resource types in the ClusterRole that your rulebooks actively watch. Granting unnecessary permissions widens the blast radius if the token is ever compromised.

-----

## Step 2: Create the ServiceAccount

**File:** [ocp/02_eda-serviceaccount.yaml](ocp/02_eda-serviceaccount.yaml)

The ServiceAccount is the machine identity the EDA controller presents when connecting to OpenShift. It is created in a dedicated namespace (e.g. `aap`) to keep it isolated from workload namespaces.  

**Apply:**

```bash
# Create the namespace first if it doesn't exist
oc create namespace aap

# Create the ServiceAccount
oc apply -f ocp/02_eda-serviceaccount.yaml
```

-----

## Step 3: Bind the ServiceAccount to the ClusterRole

**File:** [ocp/03_eda-rolebinding.yaml](ocp/03_eda-rolebinding.yaml)

The ClusterRoleBinding connects the ServiceAccount (Step 2) to the ClusterRole (Step 1), granting the identity its read-only permissions.

**Apply:**

```bash
oc apply -f ocp/eda-rolebinding.yaml
```

-----

## Step 4: Generate a Persistent Long-Lived Token

**File:** [ocp/04_eda-token-secret.yaml](ocp/04_eda-token-secret.yaml)

OpenShift does not auto-generate permanent tokens for ServiceAccounts. Since the EDA controller lives outside the cluster, you must manually create a Secret to house a permanent bearer token.

**Apply:**

```bash
oc apply -f ocp/04_eda-token-secret.yaml
```

> **⚠️ Important**
> This token does not expire automatically. Treat it like a password — store it only in AAP as a credential, rotate it periodically, and never commit the decoded value to Git.

-----

## Step 5: Extract the Token

Once the Secret is created, extract and decode the token for use in AAP:

```bash
oc get secret eda-token-secret -n aap \
  -o jsonpath='{.data.token}' | base64 --decode
```

Copy the output. You will paste it into AAP when configuring the OpenShift credential in the next section.

You can also retrieve the cluster API URL at this point:

```bash
oc whoami --show-server
```

-----

## Step 6: Label Target Namespaces  

** For namespaces provisioned through the Namespace as a Service workflow with the **EDA Governance** checkbox enabled, this label is applied automatically at provisioning time. You only need to apply it manually for namespaces provisioned outside the workflow.**

-----

## How the Rulebook Uses These Components

The table below maps each OpenShift component to the rulebook parameter that depends on it:

|Rulebook Parameter                   |OpenShift Component                            |Purpose                                                               |
|-------------------------------------|-----------------------------------------------|----------------------------------------------------------------------|
|`eda.filename.kubeconfig`            |Step 4 & 5: Token Secret                       |Provides identity and cluster endpoint to the Juniper plugin          |
|`juniper.eda.k8s` (connection)       |Step 2 & 3: ServiceAccount + ClusterRoleBinding|Identifies the incoming connection as `eda-service-account`           |
|`kind: Namespace`, `kind: Route` etc.|Step 1: ClusterRole resource list              |Verifies this account has `get/list/watch` permissions for those types|
|`label_selectors: type=eda`          |Step 6: Object labels                          |Filters events — only labelled resources generate events in the stream|

### How kubeconfig Credential Projection Works

When a Rulebook Activation runs in AAP with an OpenShift token credential assigned:

1. AAP reads the decoded OpenShift bearer token from the credential
1. AAP dynamically writes a valid `kubeconfig` file into the container runtime at activation time
1. AAP assigns the generated file path to the internal variable `{{ eda.filename.kubeconfig }}`
1. The Juniper plugin reads this file to discover the cluster API endpoint and authenticate

This means **you never hardcode cluster URLs or tokens in your rulebook YAML** — they are injected at runtime by AAP’s credential system.

-----

## Summary

```
eda-clusterrole.yaml        → What EDA can read (get/list/watch on specific resources)
eda-serviceaccount.yaml     → Who EDA is (machine identity in the aap namespace)
eda-rolebinding.yaml        → Connects identity to permissions
eda-token-secret.yaml       → Persistent token for external (out-of-cluster) access
oc label namespace ...      → Scopes event streaming to labelled resources only
```

-----

## Further Reading

- [OpenShift RBAC documentation](https://docs.openshift.com/container-platform/latest/authentication/using-rbac.html)
- [Kubernetes ServiceAccount tokens](https://kubernetes.io/docs/concepts/configuration/secret/#service-account-token-secrets)
- [AAP Credential Types — OpenShift/Kubernetes Bearer Token](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions/eda-credentials)
