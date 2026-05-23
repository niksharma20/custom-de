# Commands

## Step 1: Create the ClusterRole  
This step defines exactly what resources your rulebook can view. It is limited to read-only API access (get, list, watch).  
[eda-clusterrole.yaml](eda-clusterrole.yaml)  
```bash  
oc apply -f eda-clusterrole.yaml
```  

## Step 2: Create the ServiceAccount  
This creates the machine identity that the EDA Controller will use to log into your cluster. Note: We will create this inside a dedicated namespace named aap.  
[eda-serviceaccount.yaml](eda-serviceaccount.yaml)  
```bash  
# Create the namespace first if it doesn't exist
oc create namespace aap

# Create the ServiceAccount
oc apply -f eda-serviceaccount.yaml
```  

## Step 3: Bind the ServiceAccount to the ClusterRole  
This step connects the machine identity (Step 2) to the read-only monitoring permissions (Step 1).  
[eda-rolebinding.yaml](eda-rolebinding.yaml)  
```bash
oc apply -f eda-rolebinding.yaml
```  
## Step 4: Generate a Persistent Long-Lived Token (Crucial for Outer App Access)  
OpenShift deployments do not auto-generate permanent tokens for security compliance. Since your EDA controller lives outside OpenShift, you must manually create a Secret to house a permanent access token.  
[eda-token-secret.yaml](eda-token-secret.yaml)  
```bash  
oc apply -f eda-token-secret.yaml
```  
## Step 5: Extract the Secret to Validate and Use in AAP  
To verify everything worked and to get the password your EDA controller needs, extract the token from the secret object and decode it.  
```bash  
# Extract the API Token string
oc get secret eda-token-secret -n aap -o jsonpath='{.data.token}' | base64 --decode
```  

## Step 6: Tag Target Namespaces/Pods with type=eda  
The Juniper plugin optimizes resource consumption by ignoring untagged resources. Ensure that any project namespace or specific resource you intend to monitor contains the type=eda label.
```bash
# Label a namespace for Juniper monitoring
oc label namespace default type=eda

# Label a specific target pod
oc label pod <your-pod-name> -n default type=eda
```  
#
#
# Understanding the rulebook [openshifteda.yml](/rulebooks/openshifteda.yml) in reference to the above steps  
## 1. The Ingestion Layer: What is it doing?  
The Ingestion Layer refers to the sources: configuration block. This step instructs your custom Decision Environment container to transform from a passive background process into a live event consumer.  
```yaml
sources:
    - name: Listen for OpenShift events
      juniper.eda.k8s: # <-- The Ingestion Engine
        kinds:
          - api_version: v1
            kind: Namespace
          - api_version: v1
            kind: Route
```
When this layer executes:  
a) It maps down the juniper.eda.k8s event source plugin.  
b) It relies on the kubernetes_asyncio Python framework to build an active HTTP socket stream targeting the OpenShift API Manager.  
c) Instead of occasionally pulling data or asking for status updates, it instructs OpenShift to stream live activity data (ADDED, MODIFIED, DELETED) for those specified resource scopes.

## 2. The kubeconfig Variable: How does it get there?  
```yaml
kubeconfig: "{{ eda.filename.kubeconfig }}"
```
Ansible Automation Platform handles credential isolation via file projection.  
a) The String (eda.filename.*): When you run a Rulebook Activation inside AAP, you assign a Kubernetes/OpenShift Bearer Token credential to it.  
b) AAP reads your decoded OpenShift token, dynamically writes a valid, standard kubeconfig file on-the-fly, and mounts it into the temporary runtime environment of the container.  
c) It assigns the generated path location to the internal variable {{ eda.filename.kubeconfig }}. The Juniper plugin reads this file to learn where your OpenShift cluster lives and who it is logging in as.  

## 3. The Relation to OpenShift Implementation Steps  
Every field passed through that ingestion layer relies directly on the configuration objects created inside OpenShift. Any misconfigure in the steps, the ingestion layer will instantly fail to authorize or collect data.  
**Rulebook parameters map to your OpenShift configurations**  
```
Rulebook Parameter               OpenShift Component Matrix
---------------------------------------------------------------------------------
eda.filename.kubeconfig  ---->   Step 4 & 5: The Secret / Bearer Token string
                                 Gives the container identity and logging rights.

juniper.eda.k8s          ---->   Step 2 & 3: ServiceAccount & ClusterRoleBinding
                                 Identifies the incoming connection as "eda-service-account".

kind: Namespace          ---->   Step 1: The ClusterRole resources allowance
kind: Route                      Tells the cluster API "Yes, this account has get/list/watch 
                                 permissions for Namespaces and Routes."

event.resource.kind      ---->   Step 6: Object Metadata Labels (type=eda)
                                 Ensures OpenShift filters and pushes events to the stream 
                                 rather than ignoring the traffic profile.
```
**Summary**  
When your rulebook starts up, the Ingestion Layer reads the dynamic configuration file supplied by the kubeconfig block. It presents the ServiceAccount Token to the OpenShift API. OpenShift checks the ClusterRoleBinding to verify that this account holds the required ClusterRole permissions to watch those resources.  
If everything matches up, OpenShift begins streaming lifecycle events directly into your rulebook engine.  
