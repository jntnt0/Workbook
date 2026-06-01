Azure/
│
├── 0. Resource & Control Mechanisms/
│   ├── Azure Resource Manager
│   ├── Management Groups
│   ├── Subscriptions
│   ├── Resource Groups
│   ├── Resource Locks
│   ├── Resource Tags
│   └── Regions, Availability Zones & Paired Regions
│
├── 1. Identity & Authentication Mechanisms/
│   ├── Entra ID Directory
│   ├── Users, Groups & Devices
│   ├── MFA
│   ├── Passwordless Authentication
│   ├── SSO & Federation
│   ├── B2B External Identities
│   ├── B2C Customer Identities
│   ├── Entra Connect & Directory Sync
│   ├── Password Hash Sync & SSPR
│   └── Entra Domain Services
│
├── 2. Authorization & Access Control Mechanisms/
│   ├── RBAC
│   ├── Conditional Access
│   ├── Identity Protection
│   ├── Privileged Identity Management
│   ├── Managed Identities
│   └── App Registrations & Service Principals
│
├── 3. Governance Mechanisms/
│   ├── Azure Policy Definitions
│   ├── Policy Initiatives
│   ├── Policy Assignments & Scopes
│   ├── Policy Remediation Tasks
│   ├── Deployment Stacks
│   ├── Landing Zones
│   └── Naming & Tagging Conventions
│
├── 4. Network Forwarding & Connectivity Mechanisms/
│   ├── Virtual Networks & Subnets
│   ├── User Defined Routes
│   ├── Route Server
│   ├── VNet Peering
│   ├── Private Endpoints
│   ├── Private Link Service
│   ├── Service Endpoints
│   ├── Azure Load Balancer
│   ├── Application Gateway
│   ├── Azure Front Door
│   ├── Traffic Manager
│   ├── Azure DNS
│   ├── Private DNS Zones
│   ├── DNS Private Resolver
│   ├── VPN Gateway
│   ├── ExpressRoute
│   └── Virtual WAN
│
├── 5. Network Security & Inspection Mechanisms/
│   ├── Network Security Groups
│   ├── Application Security Groups
│   ├── Azure Firewall
│   ├── Firewall Manager
│   ├── DDoS Protection
│   ├── Web Application Firewall
│   └── Network Watcher
│
├── 6. Security Mechanisms/
│   ├── Microsoft Defender for Cloud
│   ├── Defender for Servers
│   ├── Defender for Containers
│   ├── Defender for Storage
│   ├── Defender for SQL
│   ├── Microsoft Sentinel
│   ├── Sentinel Data Connectors
│   ├── Sentinel Analytics Rules
│   ├── Key Vault
│   ├── Managed HSM
│   ├── Encryption at Rest
│   ├── Encryption in Transit
│   └── Customer-Managed Keys
│
├── 7. Compute Mechanisms/
│   ├── Virtual Machines
│   ├── VM Disks & Storage
│   ├── VM Extensions
│   ├── VM Scale Sets
│   ├── Availability Sets
│   ├── Dedicated Hosts
│   ├── App Service Plans
│   ├── Web Apps
│   ├── Deployment Slots
│   ├── Function Apps
│   ├── Durable Functions
│   ├── Azure Virtual Desktop Host Pools
│   ├── FSLogix
│   ├── Azure Batch
│   └── Azure HPC Cache
│
├── 8. Container Mechanisms/
│   ├── Azure Container Registry
│   ├── Azure Container Instances
│   ├── Azure Container Apps
│   ├── AKS Cluster
│   ├── AKS Node Pools
│   ├── AKS CNI & Networking
│   ├── AKS Ingress Controllers
│   ├── AKS Persistent Volumes & Storage Classes
│   ├── AKS RBAC & Workload Identity
│   ├── AKS Pod Security
│   ├── AKS Horizontal Pod Autoscaler
│   ├── AKS Cluster Autoscaler
│   ├── KEDA
│   ├── AKS Upgrades & Maintenance Windows
│   └── Flux & GitOps
│
├── 9. Storage Mechanisms/
│   ├── Storage Accounts
│   ├── Blob Storage
│   ├── Azure Files
│   ├── Queue Storage
│   ├── Table Storage
│   ├── Storage Access Tiers
│   ├── Lifecycle Management Policies
│   ├── Storage Redundancy
│   └── Storage Networking & Security
│
├── 10. Data Mechanisms/
│   ├── Azure SQL Database
│   ├── SQL Elastic Pools
│   ├── SQL Managed Instance
│   ├── Cosmos DB
│   ├── Cosmos DB Partitioning
│   ├── Cosmos DB Consistency Levels
│   ├── Cosmos DB Multi-Region Write
│   ├── PostgreSQL Flexible Server
│   ├── MySQL Flexible Server
│   ├── Azure Cache for Redis
│   ├── Azure Synapse Analytics
│   └── Azure Data Factory
│
├── 11. Integration Mechanisms/
│   ├── API Management
│   ├── API Management Policies
│   ├── Service Bus Queues
│   ├── Service Bus Topics & Subscriptions
│   ├── Event Grid
│   ├── Event Hubs
│   ├── Event Hubs Capture
│   ├── Logic Apps
│   ├── Azure App Configuration
│   ├── Azure SignalR & Web PubSub
│   └── Azure CDN
│
├── 12. Observability Mechanisms/
│   ├── Diagnostic Settings
│   ├── Azure Monitor Metrics
│   ├── Log Analytics Workspace
│   ├── Data Collection Rules
│   ├── Data Collection Endpoints
│   ├── KQL
│   ├── Application Insights
│   ├── Distributed Tracing
│   ├── Availability Tests
│   ├── Alert Rules
│   ├── Action Groups
│   ├── Azure Workbooks
│   ├── Azure Dashboards
│   ├── Azure Advisor
│   └── Azure Service Health
│
├── 13. Automation & IaC Mechanisms/
│   ├── ARM Templates
│   ├── Bicep
│   ├── Terraform AzureRM Provider
│   ├── Azure DevOps Pipelines
│   ├── GitHub Actions
│   ├── Automation Account
│   ├── Runbooks
│   ├── Update Management
│   └── Policy Remediation Automation
│
├── 14. Reliability Mechanisms/
│   ├── Availability Zones
│   ├── Azure Backup
│   ├── Recovery Services Vault
│   ├── Azure Site Recovery
│   ├── Geo-Redundancy & Paired Regions
│   └── Azure Chaos Studio
│
├── 15. Cost Mechanisms/
│   ├── Cost Management & Billing
│   ├── Budgets & Alerts
│   ├── Reserved Instances
│   ├── Savings Plans
│   ├── Spot Instances
│   ├── Azure Advisor Cost Recommendations
│   └── Tagging for Cost Allocation
│
├── 16. Hybrid & Migration Mechanisms/
│   ├── Azure Migrate
│   ├── Azure Data Box
│   ├── Azure Arc for Servers
│   ├── Azure Arc for Kubernetes
│   ├── Azure Arc for Data Services
│   ├── Azure Stack HCI
│   ├── Azure Stack Edge
│   └── Site Recovery for Migration  ← cross-reference: primary home is folder 14
│
├── 17. Architecture & Design Patterns/
│   ├── Well-Architected Framework
│   ├── Zero Trust Design
│   ├── Hub-Spoke Topology
│   ├── Landing Zone Design
│   ├── Microservices Patterns
│   ├── Event-Driven Architecture
│   └── Lab Walkthroughs
│
└── 18. AI & ML Mechanisms/
    ├── Azure OpenAI Deployments & Models
    ├── Azure OpenAI API & Endpoints
    ├── Prompt Engineering & RAG
    ├── Azure ML Workspace
    ├── ML Compute & Clusters
    ├── ML Pipelines
    ├── ML Model Endpoints
    ├── Azure AI Services
    ├── Azure AI Search
    ├── AI Search Indexes & Indexers
    └── Responsible AI
