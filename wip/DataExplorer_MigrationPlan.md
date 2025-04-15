# Data Explorer: Migration Plan  
...migration of infrastructure and data from one subscription to another

## Resource Inventory  
> This early-stage draft will evolve as we work through tasks.

| Resource Type | Source Subscription | Target Subscription | Infrastructure | Security | Data |
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

- Migrate existing Data Explorer-related resources from source to target subscription
- Ensure that each migrated resource is mapped to the correct subscription
- Adhere to new corporate security and governance standards

### Tasks

#### Prepare

- **Inventory**  
  - Inventory existing, source subscription resources  
  - Capture key metadata including security configurations, network settings, dependencies, etc.

- **Subscription Mapping**: Map each resource to the appropriate target subscription based on its function and governance guidelines:
  - **Platform Identity**: Centralizes directory and identity management services
  - **Platform Management**: Houses management tools such as monitoring, logging, and reporting
  - **Platform Connectivity**: Hosts networking, VPN, and connectivity services
  - **Production**: For live, business-critical systems
  - **Development**: For building and iterating new features
  - **Test**: For validating deployments and conducting pilot runs
  - **Pre-Production**: For staging and final validation before production rollout

These simplified descriptions maintain the core purpose of each subscription while keeping the language direct and easy to reference.

- **Risk Assessment**  
  - Evaluate legacy configurations and mission-critical workloads for potential migration issues  
  - Document likely risks (e.g., insufficient permissions, outdated network rules)  
  - Develop initial mitigation strategies and contingency plans


#### Assess
- **Infrastructure Suitability, Cost Planning, and Design Review**  
  - Confirm which resources align with the new landing zone requirements
  - Develop rough order-of-magnitude cost estimates using Azure pricing calculators for new or upgraded services in the target subscription.  
  - Engage finance teams to secure budget approvals based on these preliminary estimates.  
  - Identify any gaps or remediation steps needed for Terraform-based deployment.  
  - Submit documentation for review by the Architecture Review Board and the Operations Review Board.
- **Technical Architecture Documentation**  
  - Develop a comprehensive design document covering network topology, resource dependencies, authentication methods, and disaster recovery strategies.  
  - Ensure the document aligns with new landing zone requirements and compliance standards.  
  - Include detailed diagrams and metadata for review.

#### Deploy
- **Develop and Document Terraform Scripts**  
  - Build infrastructure-as-code definitions that mirror the current environment.  
  - Validate scripts in a controlled environment to ensure functional parity.  
  - Open a ServiceNow request, if required, to coordinate with the Cloud Platform team by attaching architecture and compliance documents.
- **Implement DevOps Pipelines and Integration**  
  - Establish automated pipelines for continuous integration, continuous delivery, and monitoring of infrastructure.  
  - Engage the Networking and Security Teams via ServiceNow to request any necessary firewall, virtual network, or endpoint configurations.  
  - Execute deployment in stages (Development, Test, Production) while monitoring for errors and compliance issues.  
  - Integrate automated testing checkpoints and implement rollback procedures to address any failures quickly.

#### Release
- **Post-Deployment Validation**  
  - Verify that all infrastructure components are functional in the Production environment.  
  - Conduct final checks for governance and policy adherence.  
  - Document any changes or lessons learned for future reference.

### Requirements
- **Expertise**: Identify resources with the necessary skills in Terraform, security, and networking.  
- **Documentation**: Gather and share internal references on the new landing zone, security standards, and governance policies.  
- **Access**: Confirm appropriate permissions for both the Source and target subscriptions to ensure seamless configuration and deployment.

<!-- ------------------------- ------------------------- -->

## Security

### Objective  
Ensure all migrated resources comply with corporate security and governance standards, with a focus on private connectivity, identity-based authentication, and appropriate Role-Based Access Control.

### Tasks

#### Prepare
- **Inventory**  
  - Inventory existing security configurations (such as endpoints, firewall rules, and Role-Based Access Control assignments) in the source subscription.  
  - Identify and document any use of access keys or legacy authentication methods.
- **Risk Assessment**  
  - Identify potential security gaps or vulnerabilities (for example, public endpoints or shared keys).  
  - Document likely risk scenarios, categorizing them by impact and likelihood.  
  - Develop initial mitigation strategies and assign responsibilities for resolution.

#### Assess
- **Security Architecture Review**  
  - Define required private endpoint strategies (private link, virtual network integration) for resources like Data Explorer, Storage Accounts, Synapse, etc.  
  - Collaborate with internal security teams to refine authentication standards (such as Managed Identities versus access keys).  
  - Determine the governance or compliance frameworks (ISO, NIST, or company-specific) that must be applied in the target subscription.

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

### Requirements
- **Security Engineer Expertise**: Engage someone with in-depth knowledge of Azure private endpoints and role-based access control to design and validate secure configurations.  
- **Azure AD Administrator Support**: Ensure there is access to administrators who can manage identity-related tasks, including the setup of Managed Identities and the maintenance of role-based access control.  
- **IT Compliance and Security Collaboration**: Align with internal governance, risk, and compliance groups to confirm that all newly deployed resources meet corporate and regulatory security standards.

<!-- ------------------------- ------------------------- -->

## Data

### Objective  
Migrate all critical data from Azure Data Explorer clusters in the source subscription to the target subscription without data loss or disruption to analytics operations.

### Tasks

#### Prepare
- **Inventory**  
  - Document current data volumes, structures, and dependencies within Azure Data Explorer.  
  - Identify any required data transformations or specific formats for the target subscription.  
  - Outline a preliminary migration approach (for example, using Azure Data Factory or direct Data Explorer ingestion).
- **Risk Assessment**  
  - Identify high-risk data sets (for instance, mission-critical or regulatory-compliant data) that require special handling.  
  - Note potential issues related to data ingestion, performance, or compatibility.  
  - Draft initial mitigation strategies or contingency plans for critical data.

#### Assess
- **Migration Strategy Refinement**  
  - Confirm the feasibility of the chosen migration method(s), taking into account latency, cost, and complexity.  
  - Engage analytics and compliance teams to validate requirements regarding data retention, format, or transformation.  
  - Finalize a phased migration plan that details cutover windows, testing checkpoints, and rollback options.

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