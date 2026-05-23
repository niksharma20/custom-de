## Creating Custom Credential Type for EDA, Passing as extra_vars (Environment/Memory Injection)  

```
Name: OpenShift Token Type
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
extra_vars:
  k8s_auth_host: "{{ host }}"
  k8s_auth_api_key: "{{ token }}"
```  
