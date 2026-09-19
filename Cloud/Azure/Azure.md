---
tags: [cloud, azure]
type: moc
---

# Azure

Microsoft Azure — a catalogue of services, one note each. Currently 200 services across the categories below.

## Up
- [[Cloud]]

## Services

### Compute (21)
- [[App Service]] — Fully managed platform (PaaS) to build, deploy and scale web apps, APIs and mobile backends.
- [[Azure App Service Environment]] — A fully isolated, dedicated deployment of Azure App Service into your virtual network for high-scale, secure apps.
- [[Azure Batch]] — Runs large-scale parallel and high-performance computing (HPC) batch jobs across pools of managed VMs.
- [[Azure Compute Fleet]] — Provisions and manages large, mixed fleets of Spot and on-demand VMs across sizes and zones for capacity at scale.
- [[Azure Confidential VMs]] — VMs that encrypt data in use with hardware-based trusted execution environments to protect data and code while…
- [[Azure Container Registry]] — Managed private Docker/OCI registry for storing and building container images and artifacts.
- [[Azure CycleCloud]] — An orchestration and management tool for deploying and operating HPC and big-compute clusters on Azure.
- [[Azure Dedicated Host]] — Provides physical servers dedicated to a single customer for hosting Azure VMs, aiding compliance and licensing.
- [[Azure Functions]] — Event-driven serverless compute that runs code on triggers without managing infrastructure.
- [[Azure Kubernetes Service]] — Managed Kubernetes with an Azure-operated control plane and managed node pools.
- [[Azure Modeling and Simulation Workbench]] — A managed, secure environment for engineering modeling and simulation (MODSIM) collaboration and HPC workloads.
- [[Azure Quantum]] — A cloud service providing access to quantum hardware, simulators and optimization solvers from multiple providers.
- [[Azure Red Hat OpenShift]] — A jointly engineered, fully managed Red Hat OpenShift (Kubernetes) service on Azure.
- [[Azure Spring Apps]] — A fully managed service for running Spring Boot and Spring Cloud microservices without managing infrastructure.
- [[Azure VM Image Builder]] — Automates building customized, hardened VM images from a template (built on HashiCorp Packer).
- [[Azure VMware Solution]] — Runs native VMware environments (vSphere, vSAN, NSX) on dedicated Azure infrastructure for lift-and-shift.
- [[Container Apps]] — Serverless container platform for microservices and event-driven apps built on Kubernetes and KEDA, without…
- [[Container Instances]] — Run containers on demand in seconds without managing servers or orchestrators.
- [[Service Fabric]] — A distributed-systems platform for packaging, deploying and managing scalable microservices and containers.
- [[Virtual Machine Scale Sets]] — Deploy and manage a group of identical, load-balanced VMs that autoscale with demand.
- [[Virtual Machines]] — On-demand, scalable Linux and Windows virtual machines with full control over the OS and configuration.

### Storage (16)
- [[Azure Backup]] — A centralized, managed backup service protecting VMs, databases, files and on-prem workloads.
- [[Azure Container Storage]] — A managed, container-native storage service for provisioning and managing persistent volumes for AKS at scale.
- [[Azure Data Box Gateway]] — A virtual appliance that sends data to Azure Storage over the network, presenting local SMB/NFS shares.
- [[Azure File Sync]] — Centralizes on-premises file shares in Azure Files while keeping the flexibility and performance of a local cache.
- [[Azure Files]] — Fully managed SMB and NFS file shares in the cloud, mountable from cloud and on-premises.
- [[Azure HPC Cache]] — A file-caching service that speeds access to data for high-performance computing from Blob or on-prem NAS.
- [[Azure Managed Lustre]] — A managed, fully featured Lustre parallel file system for high-performance computing workloads.
- [[Azure NetApp Files]] — Enterprise-grade, high-performance file storage powered by NetApp ONTAP, for demanding NFS/SMB workloads.
- [[Azure Storage Actions]] — A serverless service to automate data-management operations on Blob Storage objects at scale without code.
- [[Azure Storage Mover]] — A managed migration service for moving files and folders from on-prem or other clouds into Azure Storage.
- [[Blob Storage]] — Massively scalable object storage for unstructured data, with hot, cool, cold and archive tiers.
- [[Data Lake Storage]] — Blob Storage with a hierarchical namespace (Gen2) optimized for big-data analytics at scale.
- [[Elastic SAN]] — A fully managed, cloud-native storage area network (SAN) offering scalable, high-throughput block storage over…
- [[Managed Disks]] — Block-level storage volumes managed by Azure for use with VMs, in several performance tiers.
- [[Queue Storage]] — Simple, durable message queue storage for decoupling application components at scale.
- [[Table Storage]] — A NoSQL key-value/attribute store for large amounts of semi-structured, schemaless data.

### Database (14)
- [[Azure Cache for Redis]] — Fully managed, in-memory Redis-compatible cache for microsecond latency and high throughput.
- [[Azure Confidential Ledger]] — A tamper-proof, cryptographically verifiable ledger for sensitive data records, running in confidential enclaves.
- [[Azure Cosmos DB for PostgreSQL]] — Distributed PostgreSQL powered by the Citus extension for horizontal scale-out of relational workloads.
- [[Azure Managed Instance for Apache Cassandra]] — A managed service for running and scaling open-source Apache Cassandra clusters, including hybrid deployments.
- [[Azure Managed Redis]] — A next-generation, fully managed Redis service (built on Redis Enterprise) offering higher performance and…
- [[Azure SQL Database]] — Fully managed, intelligent relational database based on the latest SQL Server engine (PaaS).
- [[Cosmos DB]] — Globally distributed, multi-model NoSQL (and relational) database with turnkey global replication and low latency.
- [[Cosmos DB for Apache Gremlin]] — A graph database API on Cosmos DB using the Apache TinkerPop Gremlin query language.
- [[Cosmos DB for MongoDB]] — MongoDB-compatible database available as the request-unit API or a vCore-based cluster for familiar MongoDB…
- [[Database for MySQL]] — Fully managed MySQL (Flexible Server) with high availability, scaling and automated maintenance.
- [[Database for PostgreSQL]] — Fully managed PostgreSQL (Flexible Server) with high availability, scaling and automated maintenance.
- [[Oracle Database@Azure]] — Runs Oracle Database services (including Exadata and Autonomous Database) on OCI hardware co-located in Azure…
- [[SQL Managed Instance]] — Managed SQL Server instance with near-100% engine compatibility for easy lift-and-shift.
- [[SQL Server on Azure VMs]] — Runs full SQL Server on Azure VMs (IaaS) with the SQL IaaS Agent extension for automated management.

### Networking (21)
- [[Application Gateway]] — Layer-7 (HTTP/HTTPS) load balancer with routing, TLS termination and an optional Web Application Firewall.
- [[Application Gateway for Containers]] — A next-generation application (L7) load balancer and ingress controller optimized for Kubernetes/AKS workloads.
- [[Azure CDN]] — Caches static content at global edge locations to deliver it with low latency and high availability.
- [[Azure DNS]] — Host your DNS domains in Azure with fast, reliable name resolution using Microsoft's global network.
- [[Azure DNS Private Resolver]] — Enables querying Azure private DNS zones and resolving names between on-prem and Azure without custom DNS servers.
- [[Azure Firewall]] — Cloud-native, managed, stateful network firewall with built-in high availability and scaling.
- [[Azure Gateway Load Balancer]] — A load-balancer SKU that transparently inserts third-party network virtual appliances into the traffic path.
- [[Azure Orbital Ground Station]] — A ground-station-as-a-service to communicate with, downlink from and command satellites without owning antennas.
- [[Azure Peering Service]] — Enhances connectivity to Microsoft cloud services (Microsoft 365, Azure PaaS) over the public internet via…
- [[Azure Route Server]] — Simplifies dynamic routing between your network virtual appliances (NVAs) and your virtual network using BGP.
- [[ExpressRoute]] — Extends on-premises networks into Azure over a private, dedicated connection from a connectivity provider.
- [[Front Door]] — Global, scalable entry point that uses Microsoft's edge network for fast, secure web delivery and load balancing.
- [[Load Balancer]] — High-performance, low-latency Layer-4 (TCP/UDP) load balancing for inbound and outbound traffic.
- [[NAT Gateway]] — Provides highly scalable, resilient outbound internet connectivity for resources in a subnet using SNAT.
- [[Network Watcher]] — A suite of tools to monitor, diagnose and gain insight into your Azure network.
- [[Private Link]] — Provides private connectivity from a VNet to Azure PaaS, customer or partner services over the Microsoft backbone.
- [[Traffic Manager]] — DNS-based traffic load balancer that distributes traffic across global endpoints for availability and performance.
- [[Virtual Network]] — Your isolated private network in Azure where resources securely communicate, with subnets and routing.
- [[Virtual Network Manager]] — Centrally manages connectivity and security-rule configurations for virtual networks across subscriptions and…
- [[Virtual WAN]] — A networking service that brings connectivity, security and routing into a single, managed hub-and-spoke fabric.
- [[VPN Gateway]] — Sends encrypted traffic between an Azure VNet and on-premises or between VNets over the internet.

### Identity & Security (26)
- [[Azure Attestation]] — Remotely verifies the trustworthiness of a platform and the integrity of binaries running inside trusted…
- [[Azure Bastion]] — Provides secure RDP and SSH access to VMs directly in the portal without exposing public IPs.
- [[Azure DDoS Protection]] — Protects Azure resources from distributed denial-of-service attacks with always-on monitoring and mitigation.
- [[Azure Dedicated HSM]] — Provides physical, single-tenant HSM appliances in Azure for full administrative control of your keys.
- [[Azure Policy]] — Enforces organizational standards and assesses compliance by evaluating resources against policy rules.
- [[Firewall Manager]] — Centrally manages security policies and route management for Azure Firewall across hubs and VNets.
- [[Key Vault]] — Securely stores and controls access to secrets, keys and certificates, backed by HSMs.
- [[Managed HSM]] — A fully managed, single-tenant, FIPS 140-3 Level 3 hardware security module for high-assurance key management.
- [[Managed Identities]] — Gives Azure resources an automatically managed identity in Entra ID to access other services without storing…
- [[Microsoft Defender EASM]] — Continuously discovers and maps an organization's internet-facing assets to reveal its external attack surface.
- [[Microsoft Defender for Cloud]] — Cloud security posture management (CSPM) and workload protection (CWPP) across Azure, hybrid and multicloud.
- [[Microsoft Defender for IoT]] — Provides asset discovery, vulnerability management and threat detection for IoT and operational-technology (OT)…
- [[Microsoft Defender Threat Intelligence]] — Delivers global threat intelligence
- [[Microsoft Entra Domain Services]] — Provides managed domain services (domain join, LDAP, Kerberos/NTLM, Group Policy) without deploying domain…
- [[Microsoft Entra External ID]] — Customer identity and access management (CIAM) for securing sign-in of external users, customers and partners.
- [[Microsoft Entra ID]] — Cloud identity and access management (formerly Azure Active Directory) for authentication, SSO and authorization.
- [[Microsoft Entra ID Governance]] — Identity governance capabilities to ensure the right people have the right access to the right resources.
- [[Microsoft Entra Internet Access]] — An identity-centric Secure Web Gateway (part of Global Secure Access) that protects access to internet and SaaS…
- [[Microsoft Entra Permissions Management]] — A cloud infrastructure entitlement management (CIEM) product that discovers and right-sizes permissions across…
- [[Microsoft Entra Private Access]] — A Zero Trust Network Access (ZTNA) service (part of Global Secure Access) for secure access to private apps…
- [[Microsoft Entra Verified ID]] — A decentralized identity service for issuing and verifying digital, cryptographically secure verifiable…
- [[Microsoft Entra Workload ID]] — Secures the identities used by apps, services and workloads (non-human identities) with governance and protection.
- [[Microsoft Purview]] — A unified data governance, security and compliance platform to discover, classify, protect and manage data…
- [[Microsoft Security Copilot]] — A generative-AI security assistant that helps analysts investigate, respond to and report on security incidents.
- [[Microsoft Sentinel]] — Cloud-native SIEM and SOAR that collects, detects, investigates and responds to threats at scale.
- [[Web Application Firewall]] — Centralized protection of web apps from common exploits and vulnerabilities, deployed on Application Gateway or…

### AI (21)
- [[Azure AI Agent Service]] — A managed service (within AI Foundry) to build, deploy and scale enterprise AI agents that use tools and data.
- [[Azure AI Anomaly Detector]] — An API that detects anomalies in time-series data automatically, without requiring machine-learning expertise.
- [[Azure AI Content Safety]] — Detects and moderates harmful or unsafe content (text and images) for AI-generated and user-generated content.
- [[Azure AI Content Understanding]] — A generative-AI service that transforms unstructured multimodal content (documents, images, audio, video) into…
- [[Azure AI Custom Vision]] — Lets you build, deploy and improve custom image-classification and object-detection models with your own images.
- [[Azure AI Document Intelligence]] — Extracts text, key-value pairs, tables and structure from documents using prebuilt and custom models (formerly…
- [[Azure AI Face]] — Detects, analyzes and recognizes human faces in images (access is limited/gated for responsible use).
- [[Azure AI Foundry]] — A unified platform (formerly Azure AI Studio) to build, evaluate, deploy and manage generative-AI apps and agents.
- [[Azure AI Health Insights]] — A service providing prebuilt AI models that surface relevant insights from unstructured healthcare data.
- [[Azure AI Immersive Reader]] — An inclusively designed tool that embeds text-reading and comprehension features into applications.
- [[Azure AI Language]] — A consolidated natural-language-processing service for understanding and analyzing text.
- [[Azure AI Search]] — A managed search-as-a-service (formerly Cognitive Search) with vector, keyword and semantic retrieval for apps…
- [[Azure AI Services]] — A family of prebuilt AI APIs (formerly Cognitive Services) for vision, speech, language and decision tasks.
- [[Azure AI Speech]] — Speech-to-text, text-to-speech, translation and speaker recognition via API and SDK.
- [[Azure AI Translator]] — A neural machine translation service supporting real-time and document translation across many languages.
- [[Azure AI Video Indexer]] — Extracts deep insights from video and audio
- [[Azure AI Vision]] — Prebuilt computer-vision APIs for analyzing images and video
- [[Azure Bot Service]] — A managed environment to build, host and connect conversational bots to multiple channels.
- [[Azure Machine Learning]] — An enterprise platform to build, train, deploy and manage machine-learning models across the lifecycle (MLOps).
- [[Azure OpenAI Service]] — Provides REST access to advanced OpenAI and other foundation models with Azure security, compliance and networking.
- [[Semantic Kernel]] — An open-source SDK to orchestrate AI models, plugins and agents into applications (an AI orchestration framework).

### Analytics (12)
- [[Azure Analysis Services]] — An enterprise-grade analytical engine (tabular models) for building semantic models over data for BI.
- [[Azure Data Explorer]] — A fast, fully managed analytics service for real-time analysis of large volumes of log and telemetry data using…
- [[Azure Data Factory]] — A cloud data-integration service to orchestrate and automate data movement and transformation (ETL/ELT).
- [[Azure Data Manager for Agriculture]] — A managed platform to ingest, unify and analyze agricultural data (satellite, sensor, farm operations) for agri…
- [[Azure Data Manager for Energy]] — A fully managed, OSDU-compliant data platform for the energy industry to unify and analyze subsurface and other…
- [[Azure Databricks]] — A first-party Apache Spark-based analytics and lakehouse platform optimized for Azure.
- [[Azure HDInsight]] — A managed, open-source analytics service running frameworks such as Apache Spark, Hadoop, Hive, Kafka and HBase.
- [[Azure Stream Analytics]] — A real-time analytics engine that runs SQL-like queries over streaming data from many sources.
- [[Azure Synapse Analytics]] — An integrated analytics platform combining data warehousing, big-data (Spark) and data integration.
- [[Event Hubs]] — A big-data streaming and event-ingestion service that can receive and process millions of events per second.
- [[Microsoft Fabric]] — An all-in-one SaaS analytics platform unifying data engineering, warehousing, science, real-time and BI on OneLake.
- [[Power BI Embedded]] — Lets developers embed Power BI reports, dashboards and tiles into their own applications for customers (ISV…

### Integration (10)
- [[API Management]] — Publishes, secures, transforms and monitors APIs through a managed gateway with a developer portal.
- [[Azure Data Share]] — A service to securely share data with external organizations, with control over terms and updates.
- [[Azure Integration Environments]] — Organizes and manages Azure Integration Services resources and models business processes for tracking.
- [[Azure Notification Hubs]] — A massively scalable mobile push-notification engine that pushes to any platform from any backend.
- [[Azure Relay]] — Securely exposes services running on-premises to the cloud without opening firewall ports or changing the network.
- [[Azure SignalR Service]] — A managed service to add real-time web functionality (server-to-client push) to applications over WebSockets.
- [[Azure Web PubSub]] — A managed service for building real-time messaging web apps using WebSockets and the publish/subscribe pattern.
- [[Event Grid]] — Fully managed event routing service that delivers events from sources to handlers with pub/sub at scale.
- [[Logic Apps]] — A low-code/no-code platform to automate workflows and integrate apps, data and services with connectors.
- [[Service Bus]] — Enterprise message broker with queues and publish/subscribe topics for reliable, ordered messaging.

### Developer Tools (12)
- [[Azure API Center]] — A centralized inventory (catalog) of an organization's APIs to improve discovery, governance and consumption.
- [[Azure App Configuration]] — A managed service to centrally manage application settings and feature flags separate from code.
- [[Azure Artifacts]] — A package-management service (part of Azure DevOps) for hosting and sharing NuGet, npm, Maven, Python and…
- [[Azure Deployment Environments]] — Lets teams quickly spin up app infrastructure environments from curated, template-based catalogs.
- [[Azure Dev Box]] — Provides secure, ready-to-code cloud developer workstations preconfigured for project-specific tasks.
- [[Azure Developer CLI]] — A developer-centric command-line tool that accelerates going from local code to a running app on Azure.
- [[Azure DevOps]] — A suite of developer services for planning, source control, CI/CD, testing and artifacts.
- [[Azure DevTest Labs]] — Quickly creates managed dev/test environments with cost controls and reusable templates.
- [[Azure Load Testing]] — A fully managed load-testing service to generate high-scale load and find performance bottlenecks.
- [[Bicep]] — A domain-specific language for declaratively deploying Azure resources, transpiling to ARM templates.
- [[GitHub Advanced Security for Azure DevOps]] — Brings GitHub Advanced Security code-scanning capabilities natively into Azure DevOps repositories and pipelines.
- [[Microsoft Playwright Testing]] — A managed service that runs Playwright end-to-end browser tests at scale across parallel cloud browsers.

### Management (22)
- [[Application Insights]] — Application performance monitoring (APM) for live web apps
- [[Azure Advisor]] — A personalized recommendation engine that analyzes usage and suggests improvements across five categories.
- [[Azure Arc]] — Extends Azure management and governance to servers, Kubernetes clusters and data services anywhere
- [[Azure Automanage]] — Automatically applies and maintains best-practice configurations (backup, monitoring, security) for VMs.
- [[Azure Automation]] — Process automation, configuration management and update management for cloud and hybrid environments.
- [[Azure Business Continuity Center]] — A unified management view to monitor and govern backup and disaster-recovery protection across the estate.
- [[Azure Chaos Studio]] — A managed chaos-engineering service to inject faults and measure application resilience through controlled…
- [[Azure Deployment Stacks]] — Manages a collection of resources as a single deployable and governable unit with lifecycle and deny controls.
- [[Azure Lighthouse]] — Enables cross-tenant management so service providers and enterprises can manage many customers' resources at scale.
- [[Azure Managed Applications]] — Lets you offer cloud solutions that are easy for customers to deploy and operate, with the publisher managing…
- [[Azure Managed Grafana]] — A fully managed Grafana service for visualizing and analyzing metrics, logs and traces from many sources.
- [[Azure Managed Prometheus]] — A fully managed, Prometheus-compatible monitoring backend for collecting and querying container/infrastructure…
- [[Azure Monitor]] — Full-stack observability platform collecting metrics, logs and traces with alerting and dashboards.
- [[Azure Resource Graph]] — Fast, at-scale querying of Azure resources across subscriptions using a Kusto-like query language.
- [[Azure Resource Manager]] — The deployment and management layer for Azure
- [[Azure Resource Mover]] — Simplifies moving resources across Azure Regions with dependency awareness and validation.
- [[Azure Service Health]] — Provides personalized alerts and guidance about Azure service issues, planned maintenance and health advisories.
- [[Azure Template Specs]] — Stores ARM/Bicep templates as first-class Azure resources for sharing, versioning and controlled deployment.
- [[Azure Update Manager]] — A unified service to assess and deploy operating-system updates across Azure, on-prem and multicloud machines.
- [[Copilot in Azure]] — An AI companion embedded in the Azure portal and CLI that helps design, operate, troubleshoot and optimize Azure.
- [[Cost Management]] — Tools to monitor, allocate, forecast and optimize Azure (and AWS) spend with budgets and reports.
- [[Log Analytics]] — A log data store and query engine (KQL) at the heart of Azure Monitor for analyzing telemetry.

### Migration (4)
- [[Azure Data Box]] — A family of physical devices to transfer large amounts of data into and out of Azure when networks are impractical.
- [[Azure Migrate]] — A central hub to discover, assess and migrate servers, databases, web apps and data to Azure.
- [[Azure Site Recovery]] — Disaster recovery as a service that replicates workloads to a secondary location and orchestrates failover.
- [[Database Migration Service]] — A managed service to migrate databases to Azure data platforms with minimal downtime, online or offline.

### IoT (8)
- [[Azure Device Update for IoT Hub]] — A service to publish, target and deploy over-the-air (OTA) updates to IoT and edge devices at scale.
- [[Azure Digital Twins]] — A platform to create digital models of real-world environments (buildings, factories, grids) and reason over…
- [[Azure IoT Central]] — A managed app platform (aPaaS) to build and operate IoT solutions quickly with device templates and dashboards.
- [[Azure IoT Edge]] — Runs cloud workloads
- [[Azure IoT Hub]] — A managed service acting as a central message hub for bi-directional communication between IoT devices and the…
- [[Azure IoT Hub Device Provisioning Service]] — A helper service for IoT Hub that enables zero-touch, just-in-time provisioning of devices to the right hub at…
- [[Azure IoT Operations]] — An Arc-enabled set of services for building unified data planes at the edge for industrial/OT scenarios.
- [[Azure Sphere]] — An end-to-end IoT security solution combining a secured microcontroller (MCU), OS and a cloud security service.

### Virtual Desktop (1)
- [[Azure Virtual Desktop]] — A desktop and app virtualization service (DaaS) delivering Windows desktops and apps from Azure.

### Web (3)
- [[Azure Fluid Relay]] — A managed service powering the Fluid Framework for building real-time, multi-user collaborative applications.
- [[Azure Maps]] — Geospatial APIs and SDKs to add maps, search, routing, traffic and geofencing to web and mobile apps.
- [[Azure Static Web Apps]] — A service to build and deploy full-stack static web apps with globally distributed hosting and serverless APIs.

### Communication (1)
- [[Azure Communication Services]] — Developer APIs and SDKs to add voice, video, chat, SMS and email to applications, built on Teams infrastructure.

### Industry (4)
- [[Azure Health Bot]] — A managed platform to build compliant, AI-powered virtual health assistants with built-in medical intelligence.
- [[Azure Health Data Services]] — A managed set of health-data services (FHIR, DICOM, MedTech) to unify protected health information (PHI) in the…
- [[Azure Operator Insights]] — A managed service that ingests and analyzes massive volumes of telecom network data to deliver operational…
- [[Azure Operator Nexus]] — A hybrid, carrier-grade platform for telecom operators to deploy and manage network functions on-prem and in Azure.

### Gaming (1)
- [[Azure PlayFab]] — A complete backend platform for building, launching and operating live games (LiveOps).

### Hybrid (3)
- [[Azure Local]] — A hyperconverged infrastructure (HCI) solution (formerly Azure Stack HCI) that runs Azure services in your…
- [[Azure Stack Edge]] — A cloud-managed edge appliance that brings compute, storage and hardware-accelerated ML inference to…
- [[Azure Stack Hub]] — An integrated system that runs a consistent subset of Azure services in your own datacenter (connected or…
