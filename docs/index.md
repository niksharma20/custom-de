# Event-Driven Ansible for OpenShift  

## Architecture  
![Architecture Diagram](docs/content/Images/arch_diagram.jpg)

## Introduction

This repository contains everything needed to build, configure, and operate **Event-Driven Automation (EDA)** for OpenShift.

A Decision Environment is a container image that packages the Ansible EDA runtime, event source plugins, Python dependencies, and system libraries needed to execute rulebooks in the Ansible Automation Platform (AAP) Decision Controller. This repository provides a **custom DE** that extends the Red Hat supported base image with the [Juniper k8s.eda](https://catalog.redhat.com/en/software/collection/juniper/eda) event source plugin — enabling real-time Kubernetes and OpenShift resource event streaming directly into EDA rulebooks.

-----  
> [!WARNING]
> 
> ## Prerequisites
> We will be using a CI from Red Hat Demo to provide us the baseline Infrastrature.  
>  1) [Ansible 2.6 with EDA](https://catalog.demo.redhat.com/catalog/babylon-catalog-prod?item=babylon-catalog-prod/enterprise.aap-product-demos-cnv-aap25.prod&utm_source=webapp&utm_medium=share-link)  
>  2) Access to an **OpenShift cluster 4.18+** with cluster-admin or equivalent permissions  
>  3) The `oc` CLI authenticated to your OpenShift cluster  
>  5) A container registry accessible from your AAP instance (e.g. Quay, OpenShift internal registry) -- Optional  
-----  

## Repository Structure

|Path                      |Purpose                                                                                                  |
|--------------------------|---------------------------------------------------------------------------------------------------------|
|[rulebooks/](/rulebooks)              |EDA rulebooks — event source configuration and rules that map cluster events to AAC job templates        |
|[playbooks/](/playbooks)              |Ansible playbooks — remediation logic executed by AAC when a rulebook rule fires                         |
|[content](content)/  |All workshop content is in this directory|

-----

## Goals and Objectives

### Goals for Workshop

1. **Enable real-time Namespace governance** — move from periodic polling to event-driven reactions to cluster state changes using a custom EDA Decision Environment with the Juniper k8s event source.
1. **Bridge EDA and AAC cleanly** — rulebooks in this repo do not contain remediation logic themselves; they fire AAC job templates. This keeps decision logic (EDA) and execution logic (AAC + playbooks) cleanly separated.


### Objectives

By working through this repository, you will be able to:

- Register the DE and AAC in Ansible Automation Platform
- Configure the required OpenShift RBAC objects so EDA & AAC to watch resources and execute remidiations playbooks
- Activate a rulebook in EDA that streams live events from OpenShift
- Understand how the `kubeconfig` credential projection works in AAP
- Trigger and verify remediation playbooks via AAC job templates  

-----

## Further Reading

- [Ansible EDA documentation](https://www.ansible.com/products/event-driven-ansible)
- [Juniper k8s.eda collection](https://github.com/Juniper/k8s.eda)
- [kubernetes_asyncio Python library](https://github.com/tomplus/kubernetes_asyncio)
- [AAP Decision Environments](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/latest/html/using_automation_decisions)
- [OpenShift RBAC documentation](https://docs.openshift.com/container-platform/latest/authentication/using-rbac.html)
- [Event-Driven Ansible content by Andrew Block](https://github.com/sabre1041/sabre1041.eda)
- [demo-event-driven-ansible](https://github.com/redhat-gpte-devopsautomation/demo-event-driven-ansible/tree/main)

-----  
> [!Note]
> 
> Anthropic's Sonnet and Google's Gemini were used documentation and reserch during the creation of this workshop.
>
-----  
