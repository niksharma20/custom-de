Step 4: Launch the Rulebook Activation!
This spins up your custom DE container as a background daemon, feeds it the OpenShift token, and connects it to your repository stream.

Under Automation Decisions, click on Rulebook Activations.

Click Create rulebook activation.

Configure the launcher wizard:

Name: OpenShift Namespace Monitor Activation

Project: Select the project you made in Step 1 (Custom DE Infrastructure Git).

Rulebook: Select your target file from the dropdown list.

Decision Environment: Select your custom image profile from Step 2 (Ansible Nikhil Custom DE).

Credentials: Select both credentials you created:

Your OpenShift Cluster Token (so it can listen).

Your Red Hat Ansible Automation Platform profile (so it can execute playbooks).

Ensure the Enabled toggle is flipped on, and click Create rulebook activation.
