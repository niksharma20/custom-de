# Extra: Supporting Files and Reference Material

## Overview

The `extra/` directory contains supporting configuration, utilities, and reference material that complement the core components of the repository. These files are not directly part of the DE build or rulebook execution pipeline, but are useful for setting up, testing, and extending the platform.

-----

## Contents

### `ansible.cfg` Reference

A baseline `ansible.cfg` for configuring collection paths, default inventory, and AAP connection settings when running playbooks locally or in CI.

```ini
[defaults]
collections_path = ./collections
inventory = ./inventory/hosts.yml
stdout_callback = yaml
interpreter_python = auto_silent

[galaxy]
server_list = automation_hub, galaxy

[galaxy_server.automation_hub]
url = https://cloud.redhat.com/api/automation-hub/
auth_url = https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
token = <your-automation-hub-token>

[galaxy_server.galaxy]
url = https://galaxy.ansible.com
```

> **ℹ️ Note**
> For production use, store the Automation Hub token as an environment variable (`ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_TOKEN`) rather than in `ansible.cfg`. See the [Red Hat Developer Learning course on collections](https://developers.redhat.com/learning/learn:ansible:getting-started-ansible-content-collections/resource/resources:finding-and-installing-collections-and-using-them-playbooks?source=sso) referenced in the repository README.

-----

### Collection Requirements Reference

A `requirements.yml` for installing all necessary collections locally or in a CI pipeline:

```yaml
---
collections:
  - name: ansible.eda
  - name: community.general
  - name: kubernetes.core
  - name: https://github.com/Juniper/k8s.eda.git
    type: git
```

Install with:

```bash
ansible-galaxy collection install -r requirements.yml
```

-----

### Local Testing Utilities

The `extra/` directory may also contain scripts for:

- Generating test events manually (e.g. creating a PVC without labels to trigger a rulebook rule)
- Verifying the ServiceAccount token is valid before importing to AAP
- Checking that the custom DE image is accessible from your AAP instance

Example token validation script:

```bash
#!/bin/bash
# Validate the EDA ServiceAccount token can reach the OpenShift API
TOKEN=$(oc get secret eda-token-secret -n aap -o jsonpath='{.data.token}' | base64 --decode)
API_URL=$(oc whoami --show-server)

curl -sk \
  -H "Authorization: Bearer $TOKEN" \
  "$API_URL/api/v1/namespaces" | jq '.items[].metadata.name'
```

If this returns a list of namespaces, the token is valid and has the correct ClusterRole permissions.

-----

## Shortcuts and Optimisation Tips

The repository README notes that shortcuts and optimisation tricks will be added to this section over time. Current recommendations:

### Use `oc` aliases for common EDA operations

```bash
# Watch events on labelled namespaces in real time
alias eda-watch='oc get events -A --field-selector involvedObject.kind=Namespace -w'

# List all EDA-labelled namespaces
alias eda-ns='oc get namespaces -l type=eda'

# Check EDA governance annotations on a namespace
eda-audit() { oc get namespace "$1" -o jsonpath='{.metadata.annotations}' | jq; }
```

### Check rulebook activation health quickly

```bash
# In AAP, check activation status via API
curl -sk -u <user>:<password> \
  https://<your-aap-url>/api/eda/v1/activations/ | jq '.results[] | {name, status}'
```

### Validate DE image before registering in AAP

```bash
# Pull and inspect the DE image locally
podman pull <your-registry>/<your-org>/custom-eda-de:latest
podman run --rm <your-registry>/<your-org>/custom-eda-de:latest \
  ansible-galaxy collection list | grep juniper
```

If `juniper.eda.k8s` (or the equivalent installed name) appears in the output, the collection was built into the DE correctly.

-----

## Common Configuration Pitfalls

> **⚠️ Warning — Credentials to never expose**
> 
> - Never commit the decoded `eda-token-secret` value to Git
> - Never include `OCP_TOKEN` or `BACKSTAGE_TOKEN` values in playbook `vars:` blocks — use AAP credentials and `extra_vars` injection instead
> - Never use `verify_ssl: false` in production without understanding the security implications — use a trusted CA or add the cluster CA cert to the DE image

> **⚠️ Warning — Crucial setup steps**
> 
> - The `aap` namespace must exist before applying `eda-serviceaccount.yaml`
> - The `kubernetes_asyncio` Python package must be present in the DE image — without it the Juniper plugin will fail silently at activation startup
> - The AAC Job Template name must exactly match the `run_job_template.name` value in the rulebook — including capitalisation and spaces

-----

## Further Reading

- [Getting started with Ansible Content Collections](https://developers.redhat.com/learning/learn:ansible:getting-started-ansible-content-collections/resource/resources:finding-and-installing-collections-and-using-them-playbooks?source=sso)
- [ansible.cfg reference](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)
- [Ansible Galaxy requirements.yml](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#installing-roles-and-collections-from-the-same-requirements-yml-file)