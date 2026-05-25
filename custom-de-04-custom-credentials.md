# Custom Credential Types

## Overview

The `eda_custom_credential/` directory contains **custom credential type definitions** for AAP. These extend the built-in credential types available in Automation Decisions with organisation-specific schemas — allowing you to pass structured, encrypted configuration to rulebooks and playbooks in a standardised, reusable way.

Custom credential types are particularly useful when your rulebooks or playbooks need to interact with internal systems (such as a private Git server, an ITSM platform, or a Vault instance) that AAP does not natively support with a built-in credential type.

-----

## What Is a Custom Credential Type?

A custom credential type in AAP has three components:

|Component                 |Purpose                                                                                                                  |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------|
|**Input configuration**   |Defines the fields the user fills in when creating a credential (e.g. URL, username, token)                              |
|**Injector configuration**|Defines how the field values are exposed to the runtime — as environment variables, extra vars, or file-based credentials|
|**Credential**            |An instance of the type — the actual values stored encrypted in AAP                                                      |

Once a custom credential type is created, you can create one or more credentials from it, assign them to Job Templates or Rulebook Activations, and the values are injected at runtime without appearing in logs or playbook source.

-----

## Custom Credential Types in This Repository

### Type 1: OpenShift Token (EDA-specific)

An alternative to the built-in Kubernetes Bearer Token type, structured specifically for EDA rulebook activations with explicit field naming for the `k8s.eda` plugin.

**Input configuration:**

```yaml
fields:
  - id: ocp_api_url
    type: string
    label: OpenShift API URL
  - id: ocp_token
    type: string
    label: OpenShift Bearer Token
    secret: true
  - id: ocp_verify_ssl
    type: boolean
    label: Verify SSL
    default: true
required:
  - ocp_api_url
  - ocp_token
```

**Injector configuration:**

```yaml
extra_vars:
  ocp_api_url: "{{ ocp_api_url }}"
  ocp_verify_ssl: "{{ ocp_verify_ssl }}"
env:
  OCP_TOKEN: "{{ ocp_token }}"
```

> **ℹ️ Note**
> The `secret: true` flag on `ocp_token` ensures the value is stored encrypted in AAP’s database and never appears in job logs or API responses.

-----

### Type 2: EDA Notification Webhook

For rulebooks that need to send notifications to an external webhook (e.g. Slack, Teams, or a custom alerting endpoint) when a rule fires.

**Input configuration:**

```yaml
fields:
  - id: webhook_url
    type: string
    label: Webhook URL
    secret: true
  - id: webhook_channel
    type: string
    label: Channel or Target
required:
  - webhook_url
```

**Injector configuration:**

```yaml
extra_vars:
  webhook_channel: "{{ webhook_channel }}"
env:
  WEBHOOK_URL: "{{ webhook_url }}"
```

-----

### Type 3: Git SCM Token (for playbook-driven GitOps operations)

Used by remediation playbooks that need to open pull requests or push manifest changes back to Git as part of the remediation loop (e.g. the TTL extension PR or the NetworkPolicy restore flow).

**Input configuration:**

```yaml
fields:
  - id: git_url
    type: string
    label: Git Repository Base URL
  - id: git_token
    type: string
    label: Personal Access Token
    secret: true
  - id: git_username
    type: string
    label: Git Username
required:
  - git_url
  - git_token
  - git_username
```

**Injector configuration:**

```yaml
extra_vars:
  git_url: "{{ git_url }}"
  git_username: "{{ git_username }}"
env:
  GIT_TOKEN: "{{ git_token }}"
```

-----

## Importing Custom Credential Types into AAP

### Via the AAP UI

1. Navigate to **Automation Execution → Credential Types**
1. Click **Create credential type**
1. Enter a **Name** (e.g. `OpenShift Token (EDA)`)
1. Paste the **Input configuration** YAML into the input field
1. Paste the **Injector configuration** YAML into the injector field
1. Click **Save**

Repeat for each credential type.

### Via the AAP API or `awx` CLI

```bash
# Using the awx CLI
awx credential_types create \
  --name "OpenShift Token (EDA)" \
  --kind cloud \
  --inputs @eda_custom_credential/ocp-token-input.yml \
  --injectors @eda_custom_credential/ocp-token-injector.yml
```

-----

## Creating a Credential from a Custom Type

Once the credential type exists:

1. Navigate to **Automation Decisions → Credentials → Create credential**
1. Select your custom credential type from the **Credential Type** dropdown
1. Fill in the fields defined in the input configuration
1. Click **Save**

The credential can now be assigned to Rulebook Activations or Job Templates.

-----

## Security Best Practices

- Mark all sensitive fields (`token`, `password`, `secret`) with `secret: true` — these are encrypted at rest and never exposed in logs
- Use environment variable injection (`env:`) for secrets rather than `extra_vars:` — extra vars can appear in job output; environment variables do not
- Create one credential per system/environment — do not reuse a single credential across production and non-production environments
- Rotate credentials regularly and update the AAP credential object — playbooks and rulebooks pick up the new value on next run without code changes

-----

## Further Reading

- [AAP Custom Credential Types documentation](https://docs.ansible.com/automation-controller/latest/html/userguide/credential_types.html)
- [AWX Custom Credential Types reference](https://github.com/ansible/awx/blob/devel/docs/credentials/custom_credential_types.md)
- [AAP Credentials overview](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions/eda-credentials)