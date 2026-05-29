# [Custom Credential Types](content/custom_credential_type/custom-credentials-type.md)

## Overview

The `custom_credential_type` directory contains **custom credential type definitions** for AAP. These extend the built-in credential types available in Automation Decisions with organisation-specific schemas — allowing you to pass structured, encrypted configuration to rulebooks and playbooks in a standardised, reusable way.

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

### [OpenShift Service Account Token](content/custom_credential_type/openShift_service_account_token.md)
-----
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
- [AAP Credentials overview](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions/eda-credentials)
