# Decision Environment

## Overview

The `decision-environment.yml` file is the **build manifest** for the Custom Decision Environment. It is consumed by [`ansible-builder`](https://ansible.readthedocs.io/projects/builder/en/latest/) to produce a container image that the AAP Decision Controller (EDA) uses to execute rulebooks.

This manifest extends the Red Hat supported EDA base image with:

- The **Juniper k8s.eda** event source plugin (installed from Git)
- The **ansible.eda** and **community.general** Ansible collections
- The **kubernetes_asyncio** Python library (required by the Juniper plugin)
- Supporting Python packages and system utilities

-----

## The Manifest

```yaml
---
version: 3

images:
  base_image:
    name: registry.redhat.io/ansible-automation-platform-26/de-supported-rhel9:1.2.2-5

dependencies:
  galaxy:
    collections:
      - name: ansible.eda
      - name: community.general
      # Juniper k8s EDA event source — installed directly from Git
      - name: https://github.com/Juniper/k8s.eda.git
        type: git

  python:
    - aiokafka
    - aiohttp
    - watchdog
    # Critical dependency for the Juniper k8s event source
    - kubernetes_asyncio

  system:
    - git
    - unzip

options:
  package_manager_path: /usr/bin/microdnf
```

-----

## Field Reference

### `images.base_image`

The base container image to extend. This uses the Red Hat supported EDA Decision Environment image for RHEL 9 from AAP 2.6.

> **ℹ️ Note**
> Update the image tag (`1.2.2-5`) to match the version of AAP deployed in your environment. Always use a supported base image from `registry.redhat.io` in production. You will need valid Red Hat registry credentials to pull this image.

### `dependencies.galaxy.collections`

Three collections are installed:

|Collection                              |Source        |Purpose                                                                                                      |
|----------------------------------------|--------------|-------------------------------------------------------------------------------------------------------------|
|`ansible.eda`                           |Ansible Galaxy|Core EDA event sources and filters (webhook, kafka, alertmanager, etc.)                                      |
|`community.general`                     |Ansible Galaxy|General-purpose Ansible modules used in rulebook actions                                                     |
|`https://github.com/Juniper/k8s.eda.git`|Git           |Kubernetes/OpenShift event source plugin — streams `ADDED`, `MODIFIED`, `DELETED` events from the cluster API|


> **⚠️ Important**
> The Juniper collection is installed directly from its Git repository using `type: git`. This means the build requires outbound internet access to GitHub, or a mirrored copy of the repo accessible at build time. In air-gapped environments, host a mirror internally and update the `name` URL accordingly.

### `dependencies.python`

|Package             |Purpose                                                                                                                                  |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
|`aiokafka`          |Async Kafka client — used by `ansible.eda` Kafka event source                                                                            |
|`aiohttp`           |Async HTTP client — used by webhook and HTTP-based event sources                                                                         |
|`watchdog`          |File system event monitoring                                                                                                             |
|`kubernetes_asyncio`|**Critical** — async Python client for the Kubernetes API; required by the Juniper k8s event source to open and maintain the event stream|


> **⚠️ Important**
> `kubernetes_asyncio` is the most critical Python dependency. Without it, the Juniper plugin cannot establish the async WebSocket connection to the OpenShift API and the rulebook will fail to start.

### `dependencies.system`

|Package|Purpose                                                           |
|-------|------------------------------------------------------------------|
|`git`  |Required to clone the Juniper collection from Git during the build|
|`unzip`|Required by `ansible-galaxy` to extract collection archives       |

### `options.package_manager_path`

Points to `microdnf`, the lightweight package manager used in RHEL 9 UBI-based images. This is the correct path for `de-supported-rhel9` base images.

-----

## Building the Decision Environment

### Step 1: Install ansible-builder

```bash
pip install ansible-builder
```

### Step 2: Log in to the Red Hat registry

```bash
podman login registry.redhat.io
```

### Step 3: Build the image

```bash
ansible-builder build \
  --file decision-environment.yml \
  --tag <your-registry>/<your-org>/custom-eda-de:latest \
  --container-runtime podman
```

### Step 4: Push to your registry

```bash
podman push <your-registry>/<your-org>/custom-eda-de:latest
```

### Step 5: Register in AAP

1. Log in to your AAP instance.
1. Navigate to **Automation Decisions → Decision Environments**.
1. Click **Create decision environment**.
1. Set the **Image** field to the full image URL pushed in Step 4.
1. If your registry requires authentication, create a **Container Registry** credential first and assign it here.

-----

## Updating the DE

When you need to add a new collection, Python package, or update the base image:

1. Edit `decision-environment.yml`
1. Rebuild with `ansible-builder build`
1. Push the new image with an updated tag (e.g. `:v2` or a date-based tag)
1. Update the DE registration in AAP to point to the new image tag
1. Restart any active rulebook activations that use this DE

> **💡 Tip**
> Use a CI/CD pipeline (e.g. GitHub Actions, Tekton) to build and push the DE image automatically whenever `decision-environment.yml` changes. This keeps your DE versioned and reproducible.

-----

## Further Reading

- [ansible-builder documentation](https://ansible.readthedocs.io/projects/builder/en/latest/)
- [Juniper k8s.eda collection](https://github.com/Juniper/k8s.eda)
- [kubernetes_asyncio](https://github.com/tomplus/kubernetes_asyncio)
- [AAP Decision Environments](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions)
- [Red Hat container registry](https://registry.redhat.io)
