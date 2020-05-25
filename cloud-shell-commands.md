# Cloud Shell Commands

## IAM

```
$gcloud iam
Used to create and manipulate roles/service accounts

$gcloud iam roles
Used to copy, create, delete, list and describe roles

$gcloud iam roles list --filter="network"
Used to list all roles associated with Google Cloud Network

$gcloud iam roles describe roles/<role name>
Used to view permissions for that particular role

$gcloud iam service-accounts
Manage service accounts from Cloud Shell

$gcloud iam service-accounts list
List all service accounts. You can also view enabled/disabled accounts from here

$gcloud iam service-accounts disable <e-mail of SA>
Disables the SA

$gcloud iam service-accounts describe <e-mail of SA>
Describes more information about that particular SA

$gcloud iam roles list --filter="serviceAccount"
List all roles that have the ability to manipulate service accounts
```