# Data Explorer: Migration Plan  
...migration of infrastructure and data from one subscription to another

> Before we start, it is important to note that migration of Data Explorer can we as simple as ad-hoc steps: 1) use Azure Portal to create and secure necessary resources, 2) script copy of data from source to target instances
>
> That said, we are going for something more robust. This document is intended to define and support a more mature migration process, incorporating guidance outlined in the following documentation: 1) "CAF Az Foundations Navistar" and 2) "Internal - All Employees and Contractors - Launch Pad - Cloud Onboarding Quick Reference Guide"

## Resource Inventory  

| Type | Source Subscription | Target Subscription | Infrastructure | Security | Data |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Data Explorer | | | | | |
| Function Apps | | | | | |
| Logic Apps | | | | | |
| Managed Identities? | | | | | |
| Service Bus? | | | | | |
| Storage Accounts | | | | | |
| Synapse | | | | | |

<!-- ------------------------- ------------------------- -->

## Infrastructure

### Objectives

- Migrate existing, Data Explorer-related resources from a source subscription to a target subscription
- Ensure each migrated resource is mapped to the correct target subscription
- Deploy resources to the correct region, ensuring business continuity and disaster recovery compliance
- Adhere to new corporate security, governance, and cost management standards

### Tasks

#### Prepare

- **Inventory**
  - Inventory existing, source subscription resources  
  - Capture key metadata including security configurations, network settings, dependencies, etc.

- **Subscription Mapping**
  - Map each resource to the appropriate target subscription based on its function and governance guidelines, in accordance with the CAF Azure Foundations recommendations (sections 2.3 and 4):
    - **Platform Identity**: Centralizes directory and identity management services
    - **Platform Management**: Houses management tools such as monitoring, logging, and reporting
    - **Platform Connectivity**: Hosts networking, VPN, and connectivity services
    - **Production**: For live, business-critical systems
    - **Development**: For building and iterating new features
    - **Test**: For validating deployments and conducting pilot runs
    - **Pre-Production**: For staging and final validation before production rollout

- **Risk Assessment**
  - Evaluate legacy configurations and mission-critical workloads for potential migration issues  
  - Document likely risks {e.g., insufficient permissions, outdated network rules}
  - Develop initial mitigation strategies and contingency plans

#### Assess

- **Design, Planning, and Approval**
  - **Confirm alignment with new landing zone requirements**; examples:
      - Resources must use private endpoints and Managed Identities (no legacy access keys)
      - Resources must be deployed within approved Virtual Networks and subnets
      - Resources must follow standardized naming conventions and include required tags
  - **Prepare cost estimates** using Azure Pricing Calculators for new resources in the target subscription
  - **Prepare for Terraform**-based scripting {e.g., note custom or manual configuration requirements}
  - **Submit documentation for review** by the Architecture Review Board and the Operations Review Board (as per "Internal..." documentation: “The application team prepares the Application Design Document and submits it to (ARB) and routes to ‘C-NAV-CLOUD-ENABLEMENT’”)

- **Disaster Recovery**: Document business continuity plans (including network topology, resource dependencies, authentication methods, region pairing, backup schedules, failover procedures and supporting diagrams and metadata for automated monitoring)

- **Change Control**: Ensure that all changes to the infrastructure design or deployment are managed through a formal change control process
  - Submit proposed modifications via a ServiceNow change request and secure sign-off from the Architecture Review Board and the Operations Review Board before implementation
  - Maintain a documented audit trail of all approved changes and design adjustments for ongoing compliance and operational reference

#### Deploy

- **Terraform Scripts**: Create Infrastructure-as-Code definitions that mirror the current environment  
  - Validate scripts in a controlled environment to ensure functional parity

- **DevOps Pipelines**: Establish automated pipelines for continuous integration, continuous delivery, and monitoring of infrastructure
  - Deploy in stages (development >> test >> production)  
  - Engage Networking and Security Teams via ServiceNow to request any necessary firewall, virtual network, or endpoint configurations 

#### Release

- **Post-Deployment Validation**: Verify resources are functional and confirm policy adherence

- **Monitoring**: Configure logging for all resources and validate that monitoring dashboards and alerting mechanisms are in place for real-time issue detection and performance tracking

#### Requirements

- **Expertise**: Identify resources with the necessary skills in Terraform, security, and networking
- **Access**: Confirm appropriate permissions for source and target subscriptions
- **Cost Management**: Ensure resource deployments adhere to approved cost management practices by:
  - Using Azure pricing calculators for rough order-of-magnitude cost estimates
  - Implementing standardized resource tagging for cost allocation
  - Enforcing policies for non-production resources, such as automated shutdown schedules

<!-- ------------------------- ------------------------- -->

## Security

### Objective

- Ensure migrated resources comply with corporate security and governance standards, with a focus on private connectivity, identity-based authentication, and appropriate Role-Based Access Control

### Tasks

#### Prepare

- **Inventory**
  - Inventory existing security configurations {e.g., private endpoints, firewall rules, Role-Based Access Control assignments, etc.}  
  - Identify and document any use of access keys or legacy authentication methods
- **Risk Assessment**  
  - Identify potential security gaps or vulnerabilities (for example, public endpoints or shared keys).  
  - Document likely risk scenarios, categorizing them by impact and likelihood.  
  - Develop initial mitigation strategies and assign responsibilities for resolution.

#### Assess
- **Security Architecture Review**
  - Define required private endpoint strategies (private link, virtual network integration) for resources like Data Explorer, Storage Accounts, Synapse, etc.
  - Collaborate with internal security teams to refine authentication standards (such as Managed Identities versus access keys).
  - Determine the governance or compliance frameworks (ISO, NIST, or company-specific) that must be applied in the target subscription.

- **Governance and Change Control**
  - Ensure that all changes to security configurations and policies are managed through a formal change control process.
  - Submit proposed security modifications via a ServiceNow change request and secure sign-off from IT compliance and security review boards before implementation.
  - Maintain an audit trail of all approved changes for ongoing compliance and monitoring.

#### Deploy
- **Implement Security Controls**  
  - Configure private endpoints and update network rules as defined in the architecture review.  
  - Enforce the use of Managed Identities and eliminate direct access key usage.  
  - Assign or update Role-Based Access Control roles in line with the needs of analytics and data sharing.

#### Release
- **Security Validation and Documentation**  
  - Coordinate with IT security and compliance teams for a final review.  
  - Document final configurations, noting any exceptions or mitigation measures.  
  - Verify that ongoing compliance monitoring (alerts, logging) is active and transition support to the appropriate teams.

#### Requirements
- **Expertise**: Identify resources with the necessary skills in private endpoints and Role-Based Access Control
- **Azure AD Administrator Support**: Ensure there is access to administrators who can manage identity-related tasks, including the setup of Managed Identities and the maintenance of role-based access control.
- **IT Compliance and Security Collaboration**: Align with internal governance, risk, and compliance groups to confirm that all newly deployed resources meet corporate and regulatory security standards.
- **Access Control and Privileged Identity Management**: Implement Azure Privileged Identity Management to limit and monitor administrative access, ensuring that permissions are granted on a least-privilege, just-in-time basis.

<!-- ------------------------- ------------------------- -->

## Data

### Objective  
Migrate all critical data from Azure Data Explorer clusters in the source subscription to the target subscription without data loss or disruption to analytics operations.

### Tasks

#### Prepare
- **Inventory**: Document current data volumes, structures, and dependencies within Azure Data Explorer.  
  - Identify any required data transformations or specific formats for the target subscription.  
  - Outline a preliminary migration approach (for example, using Azure Data Factory or direct Data Explorer ingestion).
- **Risk Assessment**  
  - Identify high-risk data sets (for instance, mission-critical or regulatory-compliant data) that require special handling.  
  - Note potential issues related to data ingestion, performance, or compatibility.  
  - Draft initial mitigation strategies or contingency plans for critical data.

#### Assess
- **Migration Strategy Refinement**
  - Confirm the feasibility of chosen migration method(s), taking into account latency, cost, and complexity
  - Engage analytics and compliance teams to validate requirements for data retention, format, or transformation
  - Finalize a phased migration plan that details cutover windows, testing checkpoints, and rollback options

- **Governance and Change Control**
  - Ensure all changes to the data migration strategy are managed through a formal change control process
  - Submit proposed modifications via a ServiceNow change request (or equivalent) and secure sign-off from the appropriate governance bodies
  - Maintain a documented audit trail of all approved changes and updates for ongoing compliance and reference

#### Deploy
- **Phased Data Migration**  
  - Execute the migration in well-defined stages (for example, pilot/dry-run, partial migration, then full migration).  
  - Monitor data transfer performance, error logs, and resource usage.  
  - Collaborate with analytics teams to maintain continuity of reporting and downstream processes.

#### Release
- **Post-Migration Validation and Documentation**  
  - Perform final data integrity checks (such as record counts, sampling queries, and analytics validations).  
  - Document all migration steps, issues encountered, and resolutions for compliance and future reference.  
  - Hand off the final environment to operational teams and define processes for ongoing data refreshes.

### Requirements
- **Data Migration Specialist**: Involve a resource with technical expertise in Azure Data Explorer and migration tools (such as Azure Data Factory) to effectively plan and execute the data transfer.  
- **Analytics Team Support**: Engage analytics staff to verify data integrity, confirm usability post-migration, and assist with any domain-specific requirements.