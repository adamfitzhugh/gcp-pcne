## Kubernetes

### Kubernetes Cluster Networking
    - Node pools consist of one of more nodes (workers)
    - Kubernetes control server uses Google compute resource, and uses VPC peering to connect to nodes
    - Kubelet and Kube Proxy agents connect back to the API/Control server
    - Kube Proxy agent controls network services for each node
    - Kubernetes Nodes
        - VM instances running Kubernetes agents
        - Default Kubernetes node size is /20 - 4,096 addresses
    - Kubernetes Pods
        - Unit of deployment in Kubernetes. Runs one or more container images, storage and provides an IP address
        - Represents group of one or more app containers
        - Obtain their IP's from a Pod IP range, not from the VPC subnet
        - Default pod address size is /14 - 262,144 addresses
    - Services
        - Endpoint for application access
        - Supplies an IP, DNS and port for accessing pod services
        - Because pods come and go frequently, services provide a stable endpoint for accessing pod services
        - Obtain their IP addresses from a services IP range
        - Default address range is /20 - 4,096 addresses
    - By default each node in a Kubernetes cluster is allocated 256 IPs for pods
    - There's a limit of 110 pods per node

### GKE Cluster IP Allocation
    - Node pools can be configured across multiple zones of a region. With this, they still keep the same subnet configuration across the multiple zones
    - Regional GKE cluster defaults to 3 node pools across 3 zones and control plane masters span these zones
    - Route Based Cluster
        - Using routes to move traffic between prod network & VPC network
        - Legacy solution
        - Pods and services share a single CIDR range known as the pod address range
    - VPC Native Clusters
        - VPC cluster nodes acquire IP addresses from the VPC subnet
        - In this mode you can change the number of pod IP's assigned to the nodes
        - Uses alias IP ranges to allocate IP's to k8's resources

### GKE Private Clusters
    - Both the master and each k8 node is assigned a public IP address by default
    - Public master/public nodes is default configuration
    - In a public master/private node solution, the master requires an internal network to be able to communicate with nodes
    - In a private master/private node solution, only VM instances within the VPC can access cluster nodes
    - In a private master/private node solution, for on premise networks to reach the master, an authorized network must be configured with your local on premise subnet in order to access the master
    - Private nodes can use Cloud NAT to access the internet
    - Cloud NAT is assigned per node

### Shared VPC Clusters
    - A pod and service subnet must first be created within a shared VPC subnet in order for K8 clusters to be deployed in service projects. This must be done per cluster created
    - roles/container.hostServiceAgentUser - grants the ability to use Kubernetes engine service account of our host project to reserve the pod and service IP ranges in the host project for the service projects GKE service account
    - You can only use VPC native clusters for GKE clusters in a shared VPC setup

### Network Policy
    - By default, pods accept connections from any network source
    - You can create policy at cluster creation or after cluster creation
    - If you create a policy after cluster creation, GKE is required to recreate all cluster node pools in order to ensure they're able to run the policy process
    - After policy creation, all pods can still communicate with each other
    - Policies are built in .yaml files and are sent to the master to be deployed
    - Web to database pod traffic as example
    - Rules are stateful

LAB! Create kubenetes clusters & add network policies to them