## Network Monitoring

### Monitoring Network Operations
    - Cloud Logging collates audit logs, syslog, service logs, application logs & platform logs into a single entity
    - Once sent through the Cloud Logging API, the logs are processed by the Cloud Logs Router
    - Cloud Logs Router determines which logs to injest and which to ignore
    - Once Cloud Logs Router has processed these logs they're shipped to cloud log storage (by configuring a log sink)
    - By default there are 2 log buckets in each project where logs are stored - default bucket (30 day retention period ) & audit bucket (400 day retention period)
    - Log writer role permission in order for VM instance to write logs to Cloud Logging API
    - Cloud IAM Roles for logging
        - Logging Admin - full control over all log services
        - Logs Configuration Writer - can edit cloud log router configuration
        - Logs Viewer
        - Logs Writer
        - Private Logs Viewer
    - Admin activity, system event and access transparacy logs are free as we cannot control whether to turn these on/off in the console. All other log types have a cost associated with them
    - First 50GB of logs are free, after that it's $0.50 per month per GB
    - Pub/Sub is used to export logs to 3rd party application tools such as Splunk, Elastic Search etc

HANDS ON LAB FOR LOGGING

### VPC Flow Logs
    - For 2 VM instances communicating with each other 2 logs are generated - one for the source traffic going to the destination, and one for the destination collecting the source traffic

### Firewall Rule Logging
    - Verify/analyse the effects of firewall rules on our VPC traffic
    - Cloud Audit logs can also show you who made firewall rule changes
    - To view logs, one of the following roles needs to be assigned to the user
        - Logging viewer
        - Project owner
        - Project editor
        - Project viewer
    - By default, the default ingress and egress firewall rules cannot be viewed in logs
    - Stateful responses for allowed traffic is not logged
    - Inbound packets are logged if allowed/denied IF logging is enabled
    - Outbound packets are logged if allowed/denied IF logging is enabled

### Cloud Monitoring
    - Metrics are stored in a workspace in GCP
    - No cost to create a workspace
    - A workspace can manage up to 100 GCP projects
    - Best practice to create a project solely for the purpose of creating workspaces to provide cloud monitoring to all projects
    - A workspace can consist of dashboards, uptime checks, alerting policies, notification channels and groups
    - GCP recommends installing the monitoring agent on all VM instances
    - The monitoring agent is used to collect metrics from applications installed on the server such as apache, NGINX for example
    - In order for the server to send metrics to the workspace, we need a VM service account, monitoring metric writer role & private key credential file for the service account
    - Montoring groups are used to group resources together with the same metrics
    - Retention of metric data points is 6 weeks - there's no charge for the metrics created on GCP but metrics consumed from on-premises or AWS do incur a charge

LAB - Install monitoring script on GCP VMs and create monitoring workspace