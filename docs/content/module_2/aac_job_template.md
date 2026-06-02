# AE Project and Rulebooks Definations  

A Project in Automation Controller(AC) is a connection to a Git repository that contains your playbooks. AC syncs the repo and makes the playbooks available for Job Templates.  

> **Prerequisites**  
> Before you start you need:  
> 1) The URL of your rulebooks Git repository.  
> 2) If the repo is private — a Git credential (Personal Access Token).  
>
> I have created a public repo for the workshop with a rulebook, that we will be using for the workshop.  
> repo url: https://github.com/niksharma20/custom-de.git  

## Projects

**Creating a Project in Automation Controller (AC)**  

**Step 1:** Navigate to Projects  
```
Automation Execution → Projects → Create Project
```  
**Step 2:** Fill in the  form and save (like below)  
```
Name: Namespace Governance
Organization: Default
Execution environment: ee-ansible-techday
```  
```
Source control type: Git
Source control URL: https://github.com/niksharma20/custom-de.git
```  
```
Update revision on launch: On
```  
![Image](../../images/module/module_2/aac_projects_namespace.jpg)  

**Step3:** Verify the sync (Success looks like below).  
![Image](../../images/module/module_2/page20_aac_project.jpg)  

 Automation Controller scans the repository and indexes all .yml playbook files. Your playbooks in the playbooks/ directory will appear as selectable options when creating Job Templates.  

## Job Templates  

A Job Template defines what playbook to run, where to run it, and what credentials to use. EDA triggers these templates automatically when a rulebook rule fires.  

> **Prerequisites**  
>  ✅ Project (AAC Remediation Playbooks) — synced successfully  
>  ✅ Credential (OpenShift Cluster Token for AAC)  
>  ✅ Inventory — at minimum a localhost inventory (We will using "Demo Inventory", this should already be present)  

**Creating a Job Templates in Automation Controller**  

**Step 1:** Navigate to Job Templates  
```
Automation Execution → Templates → Create Job Template
```  
**Step 2:** Fill in the  form and save (like below)  
```
Name: OpenShift Set Resource Quota on Namespace
Job type: Run
Inventory: Demo Inventory
Project: Namespace Goverance
```  
```
Playbook: playbooks/apply_enterprise_compliance_Quotas.yml (select from the drop down)
Credential(s): "OCP Cluster Token for AAC"
Execution environment: ee-ansible-techday
```  
```
Prompt on launch: Enabled
Extra variables: 
 namespace: default-placeholder
```  
![Image](../../images/module/module_2/page23_aac_job_template_ocp.jpg)  

**Step3:** Verify (Success looks like below).  
![Image](../../images/module/module_2/page22_aac_job_template.jpg)  

**Step4:** If you would like to verify this right way. bext way is to go to your Openshift web terminal and run below command.  
```
oc apply -f - <<EOF
apiVersion: project.openshift.io/v1
kind: Project
metadata:
  name: eda-verification
  labels:
    type: eda
spec: {}
EOF
```
**Steps5:** Go to Jobs and look for the latest run of your Job Template "OpenShift Set Resource Quota on Namespace"  
Similar to the images below  
![Image](../../images/module/module_2/page24_aac_job.jpg)  

![Image](../../images/module/module_2/page25_aac_job_ocp.jpg)  

![Image](../../images/module/module_2/page26_aac_job_ocp_host.jpg)  

![Image](../../images/module/module_2/page27_aac_job_ocp_host_data.jpg)  


```
Congratulations!!!
````
