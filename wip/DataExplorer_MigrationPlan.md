# Data Explorer: Migration Plan
...migration of infrastructure and data from one subscription to another

<!-- ------------------------- ------------------------- -->

## Infrastructure

### Objective
Leverage Infrastructure-as-Code (via Terraform) to migrate existing resources from Azure Subscription 1 to Subscription 2 adhering to new corporate security and governance standards.

### Resources

| Resource Type | Security Configuration |
| :--- | :--- |
| Data Explorer | Unknown |
| Function Apps | Unknown |
| Synapse | Unknown |
| Storage Accounts | Unknown |
| Logic Apps | Unknown |
| Service Bus? | Unknown |
| Managed Identities? | Unknown |

### To-Do's
- Inventory current, Subscription1 resources (including top-level items like storage accounts, as well as second-level items like containers)
- Detail inventory with information about security, network, etc.
- Develop, document, and test deployment scripts based on current infrastructure
  - Ensure alignment with new landing zone requirements
  - Verify compliance with new subscription security policies
- Execute deployment in classic DevOps stages {i.e., dev-test-prod}

### Requirements
- Expertise: Identify available human resources that can support Terraform, security, network, etc.
- Documentation: Discover and share internal documentation for the new landing zone, etc.
- Access: Ensure access to both subscriptions (or resources with necessary access)

<!-- ------------------------- ------------------------- -->

## Security

### Objective
Ensure all migrated resources comply with new subscription security policies, emphasizing private endpoints and appropriate Role-Based Access Control.

### Key Activities
- Assess and document existing security configurations in Subscription 1.
- Configure private endpoints in Subscription 2 for ADX, Storage Accounts, and other required services.
- Transition from using access keys to Managed Identities wherever applicable.
- Define and implement RBAC according to data analytics and sharing requirements.
- Conduct security compliance validation with IT security team.

### Resources Needed
- Security engineer familiar with Azure private endpoints and RBAC
- Azure AD administrator support
- Collaboration with IT compliance and security teams

### Timeline
- Security assessment and planning: 1 week
- Private endpoints and identity management setup: 1 week
- RBAC configuration and validation: 1 week

<!-- ------------------------- ------------------------- -->

## Data

### Objective
Migrate all critical data from ADX clusters in Subscription 1 to the new Subscription 2 environment without data loss or disruption to analytics operations.

### Key Activities
- Conduct thorough data inventory including volume, structure, and dependencies.
- Determine the most suitable data migration tool/method (e.g., Azure Data Factory, direct ADX-to-ADX ingestion, etc.).
- Plan and execute data migration in phases to minimize operational impact.
- Validate data integrity post-migration through comprehensive testing.
- Document migration process and any issues encountered for audit purposes.

### Resources Needed
- Data migration specialist
- Support from analytics team to ensure data usability post-migration

### Timeline
- Data inventory and method selection: 1 week
- Phased data migration execution: 1–2 weeks
- Data validation and documentation: 1 week

## Overall Project Timeline

- **Initial Planning and Resource Inventory**: 1–2 weeks
- **Infrastructure-as-Code Preparation and Deployment**: 3–4 weeks
- **Security Configuration and Validation**: 3 weeks
- **Data Migration and Validation**: 3–4 weeks

### Total Estimated Duration: 8–10 Weeks

### Next Steps
- Complete inventory of current resources and their dependencies.
- Schedule initial meeting with Terraform and security experts to review technical details and address initial gaps.
- Initiate resource allocation and kick-off technical execution phase.