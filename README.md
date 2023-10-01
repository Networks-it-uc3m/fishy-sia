# The FISHY Secure infrastructure Abstraction 

This repository briefly describes the design of the Secure infrastructure Abstraction (**SIA**) module of the [FISHY project](https://fishy-project.eu).  It also provides information on the open-source technologies, protocols and API specificacions that have been used to implement this module. Comprehensive details concerning the SIA design and implementation are available in [deliverable D2.2](https://fishy-project.eu/library/deliverables) of FISHY.

## SIA design
The following picture outlines the SIA architectural design.

![SIA-architecture](https://github.com/Networks-it-uc3m/fishy-sia/blob/2363699be34aa5ecfdb1741dd505a727550720c3/SIA_design.png)

The SIA is responsible for the provisioning of a data-plane interface to support external and inter-domain communications within the FISHY platform (e.g., between an IoT/edge infrastructure and a cloud infrastructure, or between multiple cloud infrastructures). In addition, it controls the network access to the FISHY domains, protecting data traffic entering and leaving the domains. This functionality is mainly provided by the SIA Network Edge Device (**NED**) component of the SIA.

According to the FISHY approach, organizations are structured into different realms, based on the cybersecurity constraints, policies or rules, and realms are divided into domains, where a domain is defined as a group of assets with certain relationships (same network, infrastructure, location, etc.). 

<img src="https://github.com/Networks-it-uc3m/fishy-sia/blob/2363699be34aa5ecfdb1741dd505a727550720c3/FISHY-domains.png" width=40%>

The SIA operates at a domain level providing the proper means to interact with the NFV infrastructure resources that are available at every domain, regardless of the particular technologies that are used ([OpenStack](https://www.openstack.org), [Kubernetes](https://kubernetes.io), etc.). This functionality is provided by the SIA Northbound interface (**NBI**) and an Orchestration Function (**OF**). 

The OF is deployed at every domain, whereas the SIA NBI is a centralized component that can be used by other modules of the FISHY platform and service providers (SIA tenants). To support a proper interaction with any specific management and orchestration software stacks that exist in a domain, the SIA includes an adaptable southbound interface (**SBI**).

Another tool that is part of the SIA is the Centrally Controlled IPSec (**CCIPS**). The CCIPS goes beyond the classical point-to-point IPsec setup and provides a centralized architectural solution to control multiple IPsec endpoints or gateways. This solution is composed of a centralized E2E manager (controller) and two or more agents, based on IPsec engine in IKE-less mode (no IKE protocol is needed).  Finally, the SIA also includes a specific component for monitoring (SIA Monitor, **MON**) associated with the NED and other domain operations.

## SIA implementation
This picture succintly represents the different open-source technologies, standard APIs and protocols that have been used to implement the differente components of the SIA module. Specific details on the implementation are provided for each component in subsequent subsections.

<img src="https://github.com/Networks-it-uc3m/fishy-sia/blob/2363699be34aa5ecfdb1741dd505a727550720c3/SIA-implementation.png" width=70%>

### The SIA NBI and the OF
The SIA NBI provides the point-of-access to interact with the NFV infrastructure resources that are available at every domain. This point-of-access is offered to other FISHY blocks and components, such as the EDC, as well as to service providers. To support this functionality, the SIA NBI interfaces with the Orchestration Function (OF) available at every domain.

The SIA NBI is aligned with the Application Programming Interface (API) specification defined by ETSI for their NFV orchestrator, which is included in [ETSI NFV-SOL 005](https://portal.etsi.org/webapp/workprogram/Report_WorkItem.asp?WKI_ID=64365). This enables the SIA NBI to be consistent with standard specifications. In this regard, the NFV descriptors are based on the YANG models specified in [ETSI NFV-SOL 006](https://portal.etsi.org/webapp/workprogram/Report_WorkItem.asp?WKI_ID=63572) to ensure the interoperability and compatibility with other NFV solutions. 

Focusing on the implementation aspects, the SIA NBI is based on [HAProxy](https://www.haproxy.com). HAProxy is an open-source load-balancing software commonly used in web application architectures and content delivery networks (CDNs). It operates as a reverse proxy, receiving requests and distributing them to different backend servers according to established load-balancing rules. In addition to its main function as a load balancer, HAProxy also offers other useful features, such as the ability to protect against Distributed Denial-of-Service (DDoS) attacks and compatibility with different network protocols such as HTTP, TCP, and SSL.

On the other hand, the OF component is based on Open Source MANO ([OSM](https://osm.etsi.org)). OSM is an ETSI-hosted project that provides a Management and Orchestration (MANO) software stack aligned with the ETSI NFV specifications. A noteworthy aspect is that OSM exposes an API based on [ETSI NFV-SOL 005](https://portal.etsi.org/webapp/workprogram/Report_WorkItem.asp?WKI_ID=64365). Under the context of the project, this allows the SIA NBI to properly distribute the requests to the correspondent API of each OF available at every domain, and being compliant with the standard specification.

### The SIA Southbound interface (SBI)
As commented, the SIA must provide its functionalities regardless of the technologies that are used at every domain (e.g., ([OpenStack](https://www.openstack.org) or [Kubernetes](https://kubernetes.io)). To this purpose, the module includes an adaptable southbound interface (SBI), supporting the interaction with different NFV management and orchestration (MANO) technologies.

In this regard, the software that provides the basis for the SIA OF, [ETSI OSM](https://osm.etsi.org)), already provides support for OpenStack infrastructures, since it is an already well-established solution for NFV technologies. New platforms based on container technologies like Kubernetes (K8s) are starting to become an enticing prospect for the deployment of VNFs in cloud and edge environments, since containers provide a lightweight solution for the development and deployment of VNFs and Network Services (NS). However, OSM has limited support for the deployment of network services in K8s clusters since it only allows the deployment of VNFs as regular K8s pods. Considering the nature of the functionalities and services to be deployed in the project, it was necessary to have a solution inside K8s clusters to enable the creation and management of virtual links and networks. such that the VNFs of a network service can securely be interconnected over isolated network segments.

In this regard, a K8s oeprator has been developed within the project, [L2S-M](http://l2sm), providing the necessary tools to create isolated link-layer virtual networks inside K8s clusters. The SIA SBI in K8s clusters is implemented using a combination of OSM, L2S-M and Kubernetes, such that NFV services can be deployed on cloun native platforms. L2S-M is available as an open-source project.

### The SIA Network Edge Device (NED) overlay
The SIA architecture includes the ability to create and delete virtual link-layer networks that connect VNFs running in different domains. This way, remote VNFs can communicate as if they were connected on the same local network. This inter-domain connectivity is enabled by SDN technologies and comprises two main elements: the Network Edge Device (**NED**) **overlay network** and the Inter-Domain Connectivity Orchestrator (**IDCO**).

NEDs are programmable switching functions, implemented using [Open vSwitch](https://www.openvswitch.org). They forward traffic between domains. Each domain containing VNFs that require connectivity with other VNFs in different domains must have at least one NED. The NEDs are connected between them through point-to-point protected IP tunnels (i.e., Linux [VXLAN](https://www.rfc-editor.org/rfc/rfc7348.txt) tunnels), thereby creating a NED overlay network that interconnects all the FISHY domains. The topology of the NED overlay is established manually in our implementation.

The IDCO functions as a SDN controller, implemented as an internal application that runs within an instance of the Open Network Operating System ([ONOS](https://opennetworking.org/onos/)). All the NEDs connect to the IDCO via the OpenFlow 1.3 protocol. The IDCO is accessed through the SIA NBI using a custom HTTP REST API inspired in the [ETSI GS NFV-IFA 032](https://portal.etsi.org/webapp/workprogram/Report_WorkItem.asp?WKI_ID=68078) MSCS Management Interface that allows creating and deleting inter-domain virtual networks. The IDCO has been released as open source software in the following [repository](https://github.com/Networks-it-uc3m/snd-based-inter-cluster-communications).

### The SIA Centrally Controlled IPSec (CCIPS)
The CCIPS goes beyond the classical point-to-point IPsec setup and provides a centralized architectural solution to control multiple IPsec endpoints or gateways. The CCIPS is composed by a controller and two or more agents, deployed where the IPsec tunnel is established. In this IKE-less case, the RFC specifies a procedure on the re-keying process that is handled by the controller, when requested by the nodes.

On one side, the CCIPS controller architecture relies on a REST API as the central component to provide the NBI and establish sessions with the agents using the [NETCONF protocol](https://www.rfc-editor.org/rfc/rfc6241). On the other side, the CCIPS agents must provide an endpoint for the NETCONF protocol and manage the YANG models.

### The SIA Monitor (MON)
The MON is a SIA component that is able to monitor different parameters of multi-domain NFV environments. The implementation of the MON component is based on [Prometheus](https://prometheus.io/docs/), a well-known open-source monitoring system. The graphical user interface has been built using the [Grafana](https://grafana.com) visualization tool. 

The SIA MON component has been used as as software base in another Europen project, [CODECO](https://he-codeco.eu), where it has been re-designed to improve its flexibility and capacity to collect network performance metrics on multi-domain cloud native ecosystems. The development has been publised under an open-source license, being available on the [public CODECO repository]()).


