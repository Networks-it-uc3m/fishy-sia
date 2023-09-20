# The Secure infrastructure Abstraction 

This repository briefly describes the implementation corresponding to the Secure infrastructure Abstraction (**SIA**) module of the [FISHY project](https://fishy-project.eu). Specific details concerning the esign and implementation aspects of the SIA are available in [deliverable D2.2](https://fishy-project.eu/library/deliverables) of FISHY. 

In the following, we provide a brief description of the architectural design of this module, making reference to the implementation of it components. 

## SIA design
The following picture outlines the SIA architectural design.

![SIA-architecture](https://github.com/Networks-it-uc3m/fishy-sia/assets/36502934/f713efc7-323d-4630-b6f1-679dd6b86b6d)

The SIA is responsible for the provisioning of a data-plane interface to support external and inter-domain communications within the FISHY platform (e.g., between an IoT/edge infrastructure and a cloud infrastructure, or between multiple cloud infrastructures). In addition, it controls the network access to the FISHY domains, protecting data traffic entering and leaving the domains. This functionality is mainly provided by the SIA Network Edge Device (**NED**) component of the SIA. The SIA also includes a specific component for monitoring and telemetry information collection (SIA Monitor, **MON**) associated with the NED operations.

According to the FISHY approach, organizations are structured into different realms, based on the cybersecurity constraints, policies or rules, and realms are divided into domains, where a domain is defined as a group of assets with certain relationships (same network, infrastructure, location, etc.). 

<img src="https://github.com/Networks-it-uc3m/fishy-sia/assets/36502934/7d9d7635-5122-4ee3-880c-27011a600cb2)" width=50%>

The SIA operates at a domain level providing the proper means to interact with the NFV infrastructure resources that are available at every domain, regardless of the particular technologies that are used ([OpenStack](https://www.openstack.org), [Kubernetes](https://kubernetes.io), etc.). This functionality is provided by the SIA Northbound interface (**NBI**) and an Orchestration Function (**OF**). 

The OF is deployed at every domain, whereas the SIA NBI is a centralized component that can be used by other modules of the FISHY platform and service providers (SIA tenants). To support a proper interaction with any specific management and orchestration software stacks that exist in a domain, the SIA includes an adaptable southbound interface (**SBI**).

Another tool that is part of the SIA is the Centrally Controlled IPSec (**CCIPS**). The CCIPS goes beyond the classical point-to-point IPsec setup and provides a centralized architectural solution to control multiple IPsec endpoints or gateways. This solution is composed of a centralized E2E manager (controller) and two or more agents, based on IPsec engine in IKE-less mode (no IKE protocol is needed).

## SIA implementation
The picture below succintly represents the different open-source technologies, standard APIs and protocols that have been used to implement the SIA module.

![SIA-implementation](https://github.com/Networks-it-uc3m/fishy-sia/assets/36502934/f245357c-fee3-47d6-9b4a-9f2c266ab348)



