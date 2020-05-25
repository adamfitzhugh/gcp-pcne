# IAM

## IAM Roles

- Using Google Groups is the recommended way to assign roles to users in GCP
- Custom roles, predefined roles & primitive roles can be assigned to users

### 5 Key IAM Roles for Exam
- compute.networkAdmin - Compute Network Admin
    - 248 permissions assigned
    - You can administer GCP networks from this role
    - Unable to create or  firewall rules
    - Unable to create or delete SSL certificates
    - Unable to create or delete IAM roles

- compute.securityAdmin - Compute Security Admin
    - 52 permissions assigned
    - Able to create and delete firewall rules
    - Able to create and delete SSL certificates

- compute.xpnAdmin - Compute VPC Admin
    - 14 assigned permissions
    - Assigned to user who creates the network, sets up the host project and shares the network across multiple service projects
    - Consume shared VPC network subnets & external connections
    - Assign this role at org level
    - Able to get/set IAM policies

- compute.networkUser - Compute Network User
    - Provides access to use a shared VPC network

- compute.networkViewer - Compute Network Viewer
    - Read only access to compute engine networking resources


### To create custom IAM roles, you need the following permissions:
    - resourcemanager.projects.get
    - resourcemanager.projects.getIamPolicy
    - resourcemanager.projects.setIamPolicy


## Service Accounts
- When any GCP resource is created, a default SA is associated with that resource. This is for identity and API access
- The default SA permissions can be modified on creation of that resource
- You can assign users to act as an SA, however they must have the roles/iam.serviceAccountUser role assigned to them
- SA access the cloud API's it has permissions to in order to manipulate resources