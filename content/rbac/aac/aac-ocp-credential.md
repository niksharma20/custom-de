
### What to configure in AAP

Navigate to **Automation Decisions → Credentials → Create credential** and select **OpenShift or Kubernetes API Bearer Token** as the credential type.

> **ℹ️ Note**
> You never hardcode the token or cluster URL in the rulebook YAML. AAP’s credential projection handles this at activation time — keeping secrets out of source control entirely.

-----

## Credential 2: Red Hat Ansible Automation Platform (AAC Token)

This credential allows the EDA rulebook to fire job templates in Ansible Automation Controller when a rule condition is matched.

### What to configure in AAP

Navigate to **Automation Decisions → Credentials → Create credential** and select **Red Hat Ansible Automation Platform** as the credential type.


> **💡 Tip**
> In production, create a dedicated service account in AAC with the minimum permissions required — specifically `Execute` permission on the job templates the rulebooks need to trigger. Avoid using an admin account.

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
