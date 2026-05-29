# Decision Environment

## Overview

-----

-----

## Field Reference

### `Collections Installed in the DE Image`



|Collection                              ||Purpose                                                                                                      |
|----------------------------------------||-------------------------------------------------------------------------------------------------------------|
|`ansible.eda`                         ||Core EDA event sources and filters (webhook, kafka, alertmanager, etc.)                                      |
|`community.general`                     ||Ansible Galaxy|General-purpose Ansible modules used in rulebook actions                                                     |
|`k8s.eda.git`||Kubernetes/OpenShift event source plugin — streams `ADDED`, `MODIFIED`, `DELETED` events from the cluster API|


> **⚠️ Important**
> The Juniper collection is installed directly from its Git repository using `type: git`. This means the build requires outbound internet access to GitHub, or a mirrored copy of the repo accessible at build time. In air-gapped environments, host a mirror internally and update the `name` URL accordingly.

### `python dependencies`

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


-----

> **💡 Tip**
> Use a CI/CD pipeline (e.g. GitHub Actions, Tekton) to build and push the DE image automatically whenever `decision-environment.yml` changes. This keeps your DE versioned and reproducible.

-----

## Further Reading

- [ansible-builder documentation](https://ansible.readthedocs.io/projects/builder/en/latest/)
- [Juniper k8s.eda collection](https://github.com/Juniper/k8s.eda)
- [kubernetes_asyncio](https://github.com/tomplus/kubernetes_asyncio)
- [AAP Decision Environments](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions)
- [Red Hat container registry](https://registry.redhat.io)
