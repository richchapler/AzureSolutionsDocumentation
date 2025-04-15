# Data Explorer: Migration Plan
...migration of infrastructure and data from one subscription to another

## Resource Inventory
This is an early-stage draft that will evolve as we move through tasks in the following sections.

| Resource Type         | Infrastructure | Security | Data |
| :-------------------- | :------------- | :------- | :--- |
| Data Explorer         | TBD            | TBD      | TBD  |
| Function Apps         | TBD            | TBD      | TBD  |
| Logic Apps            | TBD            | TBD      | TBD  |
| Managed Identities?   | TBD            | TBD      | TBD  |
| Service Bus?          | TBD            | TBD      | TBD  |
| Storage Accounts      | TBD            | TBD      | TBD  |
| Synapse               | TBD            | TBD      | TBD  |

<!-- ------------------------- ------------------------- -->

## Infrastructure

### Objective
Leverage DevOps / Infrastructure-as-Code to migrate existing resources from the Source to Target Subscription adhering to new corporate security and governance standards.

### Tasks
- **Inventory Source Subscription Resources**
  - Identify all relevant Azure components (top-level resources like Storage Accounts, plus second-level items such as containers)
  - Gather key metadata (security configurations, network settings, and dependencies)
- **Develop and Document Deployment Scripts**
  - Create deployment scripts that reflect current infrastructure
  - Align scripts to new landing zone requirements and subscription security policies
  - Test and validate code to confirm functional parity with existing environment
- **Execute DevOps-Based Deployment**
  - Implement a staged approach (Development → Test → Production) leveraging Terraform
  - Monitor and adjust deployments to ensure alignment with governance and compliance standards

### Requirements
- **Expertise**  
  Identify resources with the necessary skills in Terraform, security, and networking
- **Documentation**  
  Gather and share internal references on the new landing zone, security standards, and governance policies
- **Access**  
  Confirm appropriate permissions for both Source and Target Subscriptions to enable seamless configuration and deployment

<!-- ------------------------- ------------------------- -->

## Security

### Objective
Ensure all migrated resources comply with corporate security and governance standards, prioritizing private connectivity, identity-based authentication, and appropriate Role-Based Access Control

### Tasks
- **Inventory Security Configurations**
  - Document existing network security (public/private endpoints, firewall rules) in Source Subscription
  - Record current authentication methods (use of access keys, Managed Identities)
  - Review current Role-Based Access Control assignments and permissions
- **Design Target Subscription Security Architecture**
  - Define private endpoint and network integration requirements (private link, VNet integration) for resources like Data Explorer, Storage Accounts, Synapse, and Logic Apps
  - Establish authentication standards emphasizing Managed Identities and removing access key usage
- **Implement Security Controls in Target Subscription**
  - Configure private endpoints per new landing zone guidelines
  - Deploy Managed Identities, eliminating use of access keys
  - Assign Role-Based Access Control roles tailored to analytics and data-sharing workflows
- **Validate and Document Security Compliance**
  - Coordinate validation with IT security and compliance teams
  - Document security configurations and confirm alignment with corporate security policies

### Requirements
- **Security Engineer Expertise**  
  Someone with in-depth knowledge of Azure private endpoints and Role-Based Access Control to design and validate secure configurations
- **Azure AD Administrator Support**  
  Access to an administrator or team who can manage identity-related tasks, including setting up Managed Identities and ensuring proper Role-Based Access Control assignments
- **IT Compliance & Security Collaboration**  
  Alignment with internal governance, risk, and compliance groups to confirm all newly deployed resources meet corporate and regulatory security standards

<!-- ------------------------- ------------------------- -->

## Data

### Objective
Migrate all critical data from ADX clusters in Source Subscription to Target Subscription without data loss or disruption to analytics operations

### Tasks
- **Inventory and Planning**
  - Identify data volume, structure, and dependencies within ADX
  - Determine migration approach (e.g., Azure Data Factory, direct ADX-to-ADX ingestion)
  - Outline phased data migration strategy to minimize operational impact
- **Execution**
  - Perform data migration according to defined phases
  - Monitor process for errors and performance bottlenecks
  - Coordinate with analytics teams to ensure continuity
- **Validation and Documentation**
  - Verify data integrity and completeness post-migration
  - Record migration steps, issues encountered, and resolutions for audit purposes

### Requirements
- **Data Migration Specialist**  
  A resource with technical expertise in Azure Data Explorer and migration tools (e.g., Azure Data Factory) to effectively plan and execute the data transfer
- **Analytics Team Support**  
  Involvement from analytics staff to verify data integrity, confirm usability post-migration, and assist with any domain-specific requirements