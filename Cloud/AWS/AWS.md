---
tags: [cloud, aws]
type: moc
---

# AWS

Amazon Web Services — a catalogue of services, one note each. Currently 200 services across the categories below.

## Up
- [[Cloud]]

## Services

### Compute (16)
- [[App Runner]] — Fully managed service to build, deploy and run containerized web apps and APIs directly from source or an image.
- [[Batch]] — Fully managed batch computing that dynamically provisions the optimal compute (EC2 or Fargate) to run large…
- [[EC2]] — Rent resizable virtual servers (instances) by the second, choosing an AMI, instance type, storage and network
- [[EC2 Auto Scaling]] — Automatically launches or terminates EC2 instances to maintain availability and match demand.
- [[EC2 Image Builder]] — Automates building, testing and distributing secure, up-to-date machine images (AMIs and container images).
- [[ECR]] — Private and public container image registry integrated with IAM, ECS/EKS and vulnerability scanning.
- [[ECS]] — Run and scale Docker containers on AWS using task definitions and services, on EC2 you manage or on serverless…
- [[EKS]] — Managed Kubernetes: AWS runs the highly available control plane while you run workloads on managed node groups…
- [[Elastic Beanstalk]] — Upload your code and Beanstalk provisions and manages the EC2, load balancer, scaling and health monitoring for…
- [[Fargate]] — Serverless compute for ECS and EKS: specify CPU/memory per task and AWS runs the containers with no nodes to…
- [[Lambda]] — Run functions in response to events with no servers to manage; AWS handles scaling, patching and availability,…
- [[Lightsail]] — Bundled, predictably priced virtual private servers (plus databases, load balancers and containers) aimed at…
- [[Outposts]] — Racks and servers that bring native AWS infrastructure and services on-premises for low-latency and…
- [[ParallelCluster]] — Open-source cluster-management tool that provisions and manages HPC clusters on AWS from a simple config file.
- [[Serverless Application Repository]] — A managed repository to find, publish and deploy reusable serverless applications packaged with SAM.
- [[Wavelength]] — Deploys compute and storage inside telecom 5G networks so applications reach mobile users with ultra-low latency.

### Storage (10)
- [[AWS Backup]] — Centralized service that automates and governs backups across many AWS services from a single place.
- [[DataSync]] — Online data-transfer service that moves large amounts of data to, from and between AWS storage services quickly.
- [[EBS]] — Network-attached block volumes for EC2 that persist independently of the instance lifecycle.
- [[EFS]] — Elastic, shared NFS file system that grows and shrinks automatically and can be mounted by many instances at once.
- [[File Cache]] — A fully managed, high-speed cache for datasets stored across on-prem NFS and S3, giving unified low-latency access.
- [[FSx]] — Fully managed third-party file systems: Windows File Server, Lustre, NetApp ONTAP and OpenZFS.
- [[S3]] — Object storage that holds any amount of data as objects in buckets, with 11 nines of durability and multiple…
- [[S3 Glacier]] — Archival S3 storage classes for rarely accessed data at very low cost, with configurable retrieval times.
- [[Snow Family]] — Physical devices (Snowball Edge, Snowcone) for offline data transfer and edge compute where the network is limited.
- [[Storage Gateway]] — Hybrid appliance that gives on-premises applications low-latency access to virtually unlimited AWS storage.

### Database (11)
- [[Aurora]] — AWS-built MySQL and PostgreSQL-compatible engine with a distributed storage layer, fast failover and up to 15…
- [[Aurora DSQL]] — Serverless, distributed SQL database with active-active multi-Region strong consistency and virtually unlimited…
- [[DocumentDB]] — Managed document database compatible with the MongoDB API, with storage that scales automatically.
- [[DynamoDB]] — Serverless NoSQL database delivering single-digit-millisecond latency at any scale with no servers to manage.
- [[ElastiCache]] — Managed in-memory data store using Redis/Valkey or Memcached for microsecond-latency caching.
- [[Keyspaces]] — Serverless, Apache Cassandra-compatible database that scales tables automatically with no clusters to manage.
- [[MemoryDB]] — Redis/Valkey-compatible, durable in-memory database offering microsecond reads and Multi-AZ durability.
- [[Neptune]] — Fully managed graph database supporting property-graph (Gremlin/openCypher) and RDF (SPARQL) models.
- [[RDS]] — Managed relational databases that handle provisioning, patching, backups and failover for popular engines.
- [[Redshift]] — Columnar, massively-parallel cloud data warehouse for fast SQL analytics over petabytes, with Spectrum to query…
- [[Timestream]] — Purpose-built, serverless time-series database for IoT and operational data with automatic tiering of recent vs…

### Networking (17)
- [[API Gateway]] — Fully managed service to build, secure and operate REST, HTTP and WebSocket APIs at scale.
- [[Client VPN]] — Managed, elastic OpenVPN-based remote-access VPN that lets users securely reach AWS and on-prem resources.
- [[Cloud Map]] — Service discovery for cloud resources: register application components and discover them by name with health…
- [[Cloud WAN]] — Builds, manages and monitors a unified global network across VPCs, Regions and on-prem from a central dashboard.
- [[CloudFront]] — Global content delivery network that caches content at edge locations to reduce latency, with edge compute and…
- [[Direct Connect]] — Dedicated, private network connection between your data center and AWS for consistent bandwidth and latency.
- [[Elastic Load Balancing]] — Automatically distributes incoming traffic across healthy targets in multiple AZs, with three balancer types.
- [[Global Accelerator]] — Uses the AWS global network and anycast static IPs to improve availability and performance for global users.
- [[Private 5G]] — Helps you set up and scale a private mobile network with the hardware, SIMs and software managed by AWS.
- [[PrivateLink]] — Provides private connectivity between VPCs, AWS services and on-prem without traversing the public internet,…
- [[Route 53]] — Highly available DNS, domain registration and traffic routing with health checks and failover.
- [[Route 53 Application Recovery Controller]] — Continuously verifies recovery readiness and provides highly reliable routing controls for multi-Region failover.
- [[Site-to-Site VPN]] — Creates encrypted IPsec tunnels between your on-premises network and a VPC over the internet.
- [[Transit Gateway]] — A cloud router that hub-and-spoke connects thousands of VPCs and on-premises networks through a single gateway.
- [[VPC]] — Your own isolated virtual network in AWS where you define IP ranges, subnets, routing and gateways.
- [[VPC IPAM]] — Plans, tracks and monitors IP address usage across your AWS environment from a central console.
- [[VPC Lattice]] — Application networking service that connects, secures and monitors service-to-service communication across VPCs…

### Security (26)
- [[Artifact]] — Self-service portal to download AWS compliance reports (SOC, PCI, ISO) and review/accept agreements.
- [[Audit Manager]] — Continuously collects evidence and maps it to controls to simplify audits against frameworks like SOC 2, PCI…
- [[AWS Private CA]] — A managed private certificate authority to issue and manage private certificates for your internal PKI.
- [[Certificate Manager]] — Provision, manage and deploy free public TLS certificates (and private CA) for AWS services.
- [[CloudHSM]] — Managed, dedicated, single-tenant hardware security modules (FIPS 140-3) for full control of your encryption keys.
- [[Cognito]] — Adds sign-up, sign-in and access control to apps, plus identity federation and temporary AWS credentials.
- [[Detective]] — Analyzes and visualizes security data to help investigate the root cause and scope of potential security issues.
- [[Directory Service]] — Managed Microsoft Active Directory (and alternatives) to use with AWS resources and directory-aware workloads.
- [[Firewall Manager]] — Centrally configures and manages firewall rules (WAF, Shield, Network Firewall, security groups) across an…
- [[GuardDuty]] — Continuous threat-detection service that analyzes AWS logs with machine learning to flag malicious or unusual…
- [[IAM]] — Controls who is authenticated and authorized for AWS resources through users, groups, roles and policies.
- [[IAM Access Analyzer]] — Identifies resources shared with external entities, validates policies and helps you move toward least privilege.
- [[IAM Identity Center]] — Centrally manages workforce access and single sign-on to multiple AWS accounts and business applications.
- [[Inspector]] — Automated, continuous vulnerability management that scans EC2, ECR container images and Lambda for CVEs and…
- [[KMS]] — Managed service to create and control cryptographic keys used to encrypt data across AWS and your own applications.
- [[Macie]] — Uses machine learning to discover, classify and protect sensitive data such as PII stored in Amazon S3.
- [[Network Firewall]] — Managed, stateful network firewall and intrusion prevention for your VPCs.
- [[Payment Cryptography]] — Provides payment-specific cryptographic operations and key management for card processing without managing HSMs.
- [[Secrets Manager]] — Securely stores, retrieves and automatically rotates secrets such as database credentials and API keys.
- [[Security Hub]] — Aggregates, normalizes and prioritizes security findings across accounts and runs automated security-standard…
- [[Security Lake]] — Automatically centralizes security data from AWS, SaaS and on-prem into a purpose-built data lake in OCSF format.
- [[Shield]] — Managed DDoS protection: Standard is automatic and free, while Advanced adds higher-layer defenses and support.
- [[Signer]] — Fully managed code-signing service to ensure the trust and integrity of your code and software artifacts.
- [[Verified Access]] — Provides secure, VPN-less access to corporate applications based on identity and device posture (zero trust).
- [[Verified Permissions]] — A scalable, fine-grained authorization service for your own applications using the Cedar policy language.
- [[WAF]] — Web application firewall that inspects HTTP(S) requests and blocks common exploits and unwanted traffic.

### Machine Learning (17)
- [[Amazon Q Business]] — A generative-AI assistant for the workplace that answers questions and takes actions over your enterprise data.
- [[Augmented AI]] — Makes it easy to add human review of machine-learning predictions when confidence is low or spot-checks are needed.
- [[Bedrock]] — Fully managed service to build generative-AI apps using foundation models from Amazon and third parties via one…
- [[Comprehend]] — Natural-language-processing service that extracts insights such as sentiment, entities, key phrases and…
- [[DeepRacer]] — A 1/18th-scale autonomous race car and service for learning reinforcement learning in a fun, hands-on way.
- [[Forecast]] — Time-series forecasting service that uses machine learning to predict future business outcomes.
- [[Fraud Detector]] — Managed service that builds, deploys and manages fraud-detection models using your data plus Amazon expertise.
- [[Kendra]] — Intelligent, ML-powered enterprise search that returns precise answers from across your content and data sources.
- [[Lex]] — Builds conversational interfaces (chatbots and voice bots) using the same technology as Alexa.
- [[Lookout for Vision]] — Uses computer vision to spot defects and anomalies in images for industrial quality inspection.
- [[Personalize]] — Creates real-time personalized recommendations using the same ML technology as Amazon.com.
- [[Polly]] — Turns text into lifelike speech using deep-learning voices, including neural and generative options.
- [[Rekognition]] — Adds image and video analysis to applications
- [[SageMaker]] — End-to-end platform to build, train, tune and deploy machine-learning models at scale, including generative AI.
- [[Textract]] — Automatically extracts text, forms and tables from scanned documents and images beyond simple OCR.
- [[Transcribe]] — Automatic speech recognition that converts audio and video to text, in batch or real-time streaming.
- [[Translate]] — Neural machine translation service that localizes text between many languages via API.

### Analytics (16)
- [[Athena]] — Serverless, interactive query service to analyze data in S3 (and other sources) using standard SQL.
- [[Clean Rooms]] — Lets companies securely collaborate and analyze combined datasets without sharing or copying the raw underlying…
- [[CloudSearch]] — A managed search service to set up, manage and scale a search solution for your website or application.
- [[Data Exchange]] — A marketplace to find, subscribe to and use third-party data sets in the cloud, delivered via S3, APIs or Redshift.
- [[Data Firehose]] — Fully managed service to reliably load streaming data into data lakes, warehouses and analytics services.
- [[DataZone]] — A data-management service to catalog, discover, share and govern data across the organization with a business…
- [[EMR]] — Managed big-data platform to run Apache Spark, Hadoop, Hive and Presto on scalable clusters (EC2, EKS or…
- [[FinSpace]] — A data-management and analytics service purpose-built for the financial-services industry, with time-series…
- [[Glue]] — Serverless data-integration service for cataloging, cleaning and transforming data (ETL/ELT).
- [[Glue DataBrew]] — A visual data-preparation tool that lets analysts clean and normalize data without writing code.
- [[Kinesis]] — Ingests and processes real-time streaming data at scale across a family of services.
- [[Lake Formation]] — Simplifies building, securing and governing data lakes on S3 with centralized, fine-grained permissions.
- [[Managed Service for Apache Flink]] — Fully managed Apache Flink for transforming and analyzing streaming data in real time (formerly Kinesis Data…
- [[MSK]] — Fully managed Apache Kafka for building real-time streaming data pipelines and applications.
- [[OpenSearch Service]] — Managed OpenSearch (and legacy Elasticsearch) for search, log analytics and observability, with Dashboards.
- [[QuickSight]] — Cloud-native, serverless business-intelligence service for interactive dashboards and ML-powered insights.

### Application Integration (9)
- [[Amazon MQ]] — Managed message broker for Apache ActiveMQ and RabbitMQ, easing migration of apps that use standard messaging…
- [[AppFlow]] — No-code service to securely transfer data between SaaS applications and AWS services on a schedule or event.
- [[AppSync]] — Managed GraphQL (and Pub/Sub) service that connects apps to data sources with real-time and offline capabilities.
- [[B2B Data Interchange]] — Automates the transformation of EDI documents into common data formats to modernize business-partner integrations.
- [[EventBridge]] — Serverless event bus that routes events between AWS services, SaaS applications and your own apps.
- [[MWAA]] — Managed Apache Airflow to author, schedule and monitor data-pipeline workflows without operating the…
- [[SNS]] — Publish/subscribe messaging that pushes messages to many subscribers such as queues, functions, HTTP endpoints,…
- [[SQS]] — Fully managed message queues that decouple producers and consumers and buffer work reliably.
- [[Step Functions]] — Coordinates multiple AWS services into serverless workflows defined as state machines.

### Developer Tools (12)
- [[Amazon Q Developer]] — Generative-AI coding assistant that answers questions, writes and transforms code, and helps operate AWS.
- [[AWS SAM]] — An open-source framework and CLI for building serverless applications with a concise template that transforms…
- [[Cloud Development Kit]] — Defines cloud infrastructure in familiar programming languages and provisions it via CloudFormation.
- [[CloudShell]] — A browser-based shell, pre-authenticated with your console credentials, with AWS CLI and common tools preinstalled.
- [[CodeArtifact]] — Fully managed artifact repository for storing and sharing software packages (npm, PyPI, Maven, NuGet and more).
- [[CodeBuild]] — Fully managed continuous-integration service that compiles source, runs tests and produces deployable artifacts.
- [[CodeCatalyst]] — Unified DevOps service to plan, build and deliver software with integrated projects, repos, CI/CD and dev…
- [[CodeDeploy]] — Automates application deployments to EC2, on-prem, ECS and Lambda with controlled rollouts and rollback.
- [[CodeGuru]] — Uses ML to find code issues and costly lines: automated code reviews (Reviewer) and runtime performance…
- [[CodePipeline]] — Fully managed continuous-delivery service that models, visualizes and automates release pipelines.
- [[Infrastructure Composer]] — A visual designer to build serverless and infrastructure applications and generate CloudFormation/SAM templates.
- [[X-Ray]] — Distributed tracing service that helps analyze and debug the behavior and performance of applications and…

### Management (22)
- [[AWS Health]] — Provides ongoing visibility into the health of your AWS resources, services and accounts, with personalized…
- [[Chatbot]] — Lets you monitor and operate AWS resources from Slack and Microsoft Teams, receiving alerts and running commands.
- [[CloudFormation]] — Provision and manage AWS infrastructure as code using declarative JSON/YAML templates and stacks.
- [[CloudTrail]] — Records account activity and API calls for governance, compliance and security auditing.
- [[CloudWatch]] — Observability service that collects metrics, logs and events, with alarms, dashboards and automated actions.
- [[Compute Optimizer]] — Analyzes utilization metrics and recommends optimal, right-sized resource configurations to cut cost and…
- [[Config]] — Records resource configurations over time and evaluates them against compliance rules.
- [[Control Tower]] — Automates setup of a secure, multi-account landing zone with pre-configured guardrails and an account factory.
- [[DevOps Guru]] — Uses ML to detect operational issues and anomalies before they impact customers, with actionable insights.
- [[Fault Injection Service]] — A managed chaos-engineering service to run fault-injection experiments and improve application resilience.
- [[License Manager]] — Centrally manages software licenses from vendors to track usage, enforce limits and reduce compliance risk.
- [[Managed Grafana]] — Fully managed Grafana for querying, visualizing and alerting on metrics, logs and traces from many sources.
- [[Managed Service for Prometheus]] — Prometheus-compatible, serverless monitoring for container and infrastructure metrics at scale.
- [[Organizations]] — Centrally manage and govern multiple AWS accounts with consolidated billing and policy controls.
- [[Proton]] — A deployment service for platform teams to define standardized infrastructure templates for developers to…
- [[Resilience Hub]] — Assesses and improves the resilience of applications against recovery objectives (RTO/RPO) with recommendations.
- [[Resource Access Manager]] — Securely shares AWS resources across accounts and within your organization without duplicating them.
- [[Service Catalog]] — Lets organizations curate and govern a catalog of approved IT products for users to self-service deploy.
- [[Service Quotas]] — A central place to view and manage your AWS service quotas (limits) and request increases.
- [[Systems Manager]] — A suite for operating AWS and on-prem fleets: run commands, patch, manage parameters and open shells.
- [[Trusted Advisor]] — Inspects your AWS environment and recommends improvements across cost, performance, security, fault tolerance…
- [[Well-Architected Tool]] — Helps you review workloads against AWS best practices (the Well-Architected Framework) and track improvements.

### Migration & Transfer (5)
- [[Application Migration Service]] — Lift-and-shift (rehost) service that replicates on-prem or cloud servers to AWS with minimal downtime.
- [[Database Migration Service]] — Migrates databases to AWS with minimal downtime, including homogeneous and heterogeneous (with schema…
- [[Elastic Disaster Recovery]] — Scalable disaster recovery (DRS) that continuously replicates servers to AWS for fast, low-cost failover.
- [[Migration Hub]] — Central place to discover on-prem resources, plan migrations and track progress across AWS migration tools.
- [[Transfer Family]] — Fully managed SFTP, FTPS, FTP and AS2 file transfer directly into and out of S3 and EFS.

### IoT (7)
- [[IoT Core]] — Connects billions of devices to AWS and routes messages between them and cloud services securely at scale.
- [[IoT Device Defender]] — Audits IoT configurations and continuously monitors device fleets to detect anomalies and security issues.
- [[IoT Device Management]] — Onboard, organize, monitor and remotely manage large fleets of IoT devices throughout their lifecycle.
- [[IoT FleetWise]] — Makes it easier to collect, transform and transfer vehicle data to the cloud in near-real time for…
- [[IoT Greengrass]] — Edge runtime that brings local compute, messaging, ML inference and data sync to IoT devices, even offline.
- [[IoT SiteWise]] — Collects, stores, organizes and monitors industrial equipment data at scale to power operational insights.
- [[IoT TwinMaker]] — Builds digital twins of real-world systems by connecting existing data sources to 3D visualizations.

### Media (7)
- [[Deadline Cloud]] — A fully managed render-farm-management service to set up, deploy and scale rendering pipelines for VFX and design.
- [[Interactive Video Service]] — Managed live streaming (IVS) for building low-latency, interactive video experiences into apps quickly.
- [[MediaConnect]] — A transport service for live video that reliably and securely moves high-value content into, through and out of…
- [[MediaConvert]] — File-based video transcoding service that converts content into formats for broadcast and multiscreen delivery.
- [[MediaLive]] — Broadcast-grade live video encoding service that turns live sources into streams for TVs and connected devices.
- [[MediaPackage]] — Just-in-time packaging and origination that prepares and protects live and VOD video for delivery.
- [[MediaTailor]] — Personalized ad insertion and channel assembly for streaming video, with server-side ad insertion (SSAI).

### Front-End Web & Mobile (3)
- [[Amplify]] — Full-stack platform to build, ship and host web and mobile apps, with hosting, backend and libraries.
- [[Device Farm]] — Tests mobile and web apps on real devices in the AWS cloud, and enables remote device access.
- [[Location Service]] — Adds maps, places, routing, geofencing and asset tracking to applications using trusted data providers.

### End User Computing (3)
- [[AppStream 2.0]] — Streams individual desktop applications from the cloud to any device through a web browser.
- [[WorkSpaces]] — Fully managed, persistent virtual desktops (DaaS) for Windows and Linux, accessible from anywhere.
- [[WorkSpaces Secure Browser]] — A managed, fully isolated browser that streams web content so corporate data never lives on the user's device.

### Business Applications (7)
- [[Amazon Connect]] — Cloud contact center that lets you set up omnichannel customer support (voice, chat, tasks) at scale.
- [[Chime SDK]] — Developer building blocks to add real-time audio, video, screen sharing and messaging to your own applications.
- [[Pinpoint]] — A multichannel customer-engagement service for targeted campaigns and transactional messages (email, SMS, push,…
- [[SES]] — Scalable, cost-effective email service for transactional, marketing and bulk email sending and receiving.
- [[Supply Chain]] — A cloud application that unifies supply-chain data and provides ML-powered insights to lower costs and mitigate…
- [[Wickr]] — An end-to-end encrypted service for secure messaging, calling, file transfer and collaboration with strong…
- [[WorkMail]] — Managed, secure business email and calendaring with support for existing desktop and mobile clients.

### Cost Management (4)
- [[Budgets]] — Sets custom cost and usage budgets and alerts you when spend or utilization crosses defined thresholds.
- [[Cost and Usage Report]] — Delivers the most detailed, line-item cost and usage data to S3 for deep analysis (now via Data Exports).
- [[Cost Explorer]] — Visualizes, understands and forecasts your AWS spend and usage over time with filtering and grouping.
- [[Savings Plans]] — A flexible pricing model offering lower prices in exchange for a committed hourly spend over 1 or 3 years.

### Industry (4)
- [[HealthImaging]] — A HIPAA-eligible service to store, transform and analyze medical imaging data (DICOM) at petabyte scale.
- [[HealthLake]] — A HIPAA-eligible service to store, transform, query and analyze health data at scale in the FHIR standard.
- [[HealthOmics]] — Helps healthcare and life-science organizations store, query and analyze genomic, transcriptomic and other…
- [[Monitron]] — An end-to-end system (sensors, gateway and app) that uses ML to detect abnormal machinery behavior for…

### Game Tech (3)
- [[Braket]] — A managed quantum computing service to explore, build and run quantum algorithms on simulators and real QPUs.
- [[GameLift]] — Dedicated server hosting and matchmaking for session-based multiplayer games at scale.
- [[Ground Station]] — Fully managed ground-station-as-a-service to control satellites and ingest their data without owning antennas.

### Blockchain (1)
- [[Managed Blockchain]] — Creates and manages scalable blockchain networks and provides node access to public blockchains.
