```mermaid
graph TD
    %% Define Styles and Colors
    classDef k8s fill:#326ce5,stroke:#fff,stroke-width:2px,color:#fff;
    classDef aap fill:#ee0000,stroke:#fff,stroke-width:2px,color:#fff;
    classDef dc fill:#4A5568,stroke:#fff,stroke-width:2px,color:#fff;
    classDef edge stroke:#718096,stroke-width:2px,stroke-dasharray: 5 5;

    subgraph K8S [Kubernetes / OpenShift]
        API[API Server Events]
    end
    class K8S,API k8s;

    subgraph AAP [Ansible Automation Platform]
        DE[Decision Environment<br>juniper.eda.k8s]
        AC[Automation Controller<br>Executes Playbooks]
    end
    class AAP,DE,AC aap;

    subgraph DC [Data Center Management]
        JA[Juniper Apstra<br>HPE Apstra Data Center Director]
        LS[Leaf/Spine Fabric<br>Multi-vendor Network Switches]
    end
    class DC,JA,LS dc;

    %% Flows
    API -->|Encrypted Event Stream<br>via kubernetes_asyncio| DE
    DE -->|Evaluates Rulebooks| AC
    AC -->|API Calls / Netconf| JA
    JA -->|Physical Switch Provisioning| LS
