# AAC Infra Definations  

Ansible Automation Controller(AAC) needs a credential to authenticate to OpenShift so the remediation playbooks can execute kubernetes.core.k8s tasks against the cluster.

> **Prerequisites**  
> 1) The OpenShift cluster API URL  
>      `oc whoami --show-server`  
> 2) The decoded bearer token from the AAC ServiceAccount  
>       `oc get secret aac-token-secret -n aap -o jsonpath='{.data.token}' | base64 --decode`  

## Creating AAC Credentials for OpenShift  
**Step1:** Log in to AAP  
**Step2:** Navigate to Credentials  
```
Automation Controller → Infrastructure → Credentials → Create Credential  
```  
**Step3:** Fill in the credential form and save (like below)  
```
Name: OCP Cluster Token for AAC
Organization: Default
```  
```
Credential Type: OpenShift or Kubernetes API Bearer Token
```  
```
OpenShift or Kubernetes API Endpoint: https://api.<your-cluster-domain>:6443 (this is part of the Prerequisites).
Bearer Token: (this is part of the Prerequisites)
```  
```
Verify SSL: On
```  
![Image](../../images/module/module_2/page18_aac_credential.jpg)  


## Execution Environement  

Creating the Execution Environment in AAP

> **Prerequisites**  
> The custom EE image built and pushed to your registry  
> Registry credentials if your registry requires authentication or make the image public.  
> 

**Step1:** Navigate to Execution Environments  
```
Automation Decisions → Infrastructure → Execution Environments → Create Execution Environment
```  
**Step2:** Fill in the form and save (like below)  
```
Name: ee-ansible-techday
Organization: Default
```  
```
Image: quay.io/rhn_support_idhaoui/ee-custom/ee-ansible-ssa:latest ( This public image, custom built for the workshop)
```  
```
Pull: Only pull the image if not present before running
```  

![Image 1](../../images/module/module_2/aac_ee_ansible_techday.jpg)  

![Image 2](../../images/module/module_2/page19_aac_ee.jpg)  

-----  

### Important Collections part EE Image  


### Python dependencies part EE Image  

-----  
