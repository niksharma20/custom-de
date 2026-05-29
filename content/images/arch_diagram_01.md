```mermaid
graph TD
    %% Define Styles and Colors
    classDef ocp fill:#326ce5,stroke:#fff,stroke-width:2px,color:#fff;
    classDef aap fill:#ee0000,stroke:#fff,stroke-width:2px,color:#fff;
    classDef ns fill:#4A5568,stroke:#fff,stroke-width:2px,color:#fff;

    subgraph Cluster [OpenShift Platform Cluster]
        API[OpenShift API Server]
        
        subgraph NS [Governed Namespace]
            STATE["Desired State Restored<br>[eda.ansible.com/last-remediated](https://eda.ansible.com/last-remediated): timestamp"]
        end
    end
    class API ocp;
    class NS,STATE ns;

    subgraph AAP [Ansible Automation Platform]
        DE[Decision Environment<br>juniper.eda.k8s]
        AC[Automation Controller AAC]
        PB[Remediation Playbook<br>kubernetes.core]
    end
    class AAP,DE,AC,PB aap;

    %% Workflow Flows
    API -->|Encrypted async event stream<br>Filtered by label: type=eda<br>Powered by: kubernetes_asyncio| DE
    DE -->|"Event payload evaluated against rulebook<br>Conditions: event.type, event.resource.kind..."| AC
    AC -->|"Matched rule fires run_job_template<br>Extra vars: namespace, resource_name..."| PB
    PB -->|"Patches / restores / annotates resource<br>Writes NIS2/CIS compliance annotation"| STATE
    
    %% The Loop Back
    STATE -.->|New event triggers<br>next watch cycle| API
