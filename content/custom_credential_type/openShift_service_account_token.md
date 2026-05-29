## Creating Custom Credential Type for EDA, Generating a file: Kubeconfig (Secure File Projection)  

```
Name: OpenShift Service Account Token
Description: For mapping OpenShift Bearer tokens into rulebooks
```

**Input Configuration**  
```yaml
fields:
  - id: host
    type: string
    label: OpenShift API Server URL
  - id: token
    type: string
    label: Bearer Token
    secret: true
required:
  - host
  - token
```  
**Injector Configuration**
```yaml
file:
  template.kubeconfig: |
    apiVersion: v1
    kind: Config
    clusters:
      - cluster:
          insecure-skip-tls-verify: true
          server: "{{ host }}"
        name: openshift-cluster
    contexts:
      - context:
          cluster: openshift-cluster
          user: eda-runner
        name: eda-context
    current-context: eda-context
    users:
      - name: eda-runner
        user:
          token: "{{ token }}"
```  
