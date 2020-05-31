# Networking

### Global Network (VPC)
    - Premium Network Tier - Ingress traffic enters Googles network at the location nearest to them. Egress traffic is sent through Google's backbone network leaving at the Global POP edge closest to the user
    - Standard Network Tier - Ingress traffic enters Google's network through peering, ISP or transit networks in the region the resources are deployed. Egress traffic is sent via peering or transit network local to where the traffic was originated from

### Making a VPC
    - You can assign up to 20 secondary IP ranges to a subnet
    - Secondary subnets are used for allocating different IP ranges on a VM instance for the different services they provide
    - Automatic (subnet expands to /16) vs Custom subnet (subnet expands to /8) networks
    - You can change your network from automatic to custom at a later date but CANNOT change from custom to automatic
    - Default network is automatically created when you create a VPC
    - Google reserves 4 addresses per primary subnet
        - Network address (.0)
        - Gateway address (which is .1)
        - Second-to-last address (.254 - for potential future use)
        - Broadcast address (.255)
    - For secondary subnets, just the network address is reserved
    - When a subnet is created, routes are automatically created within GCP to route between those subnets
    - Addresses are automatically assigned to VM's from their respective subnets by the GCP metadata server

### VPC Routes
    - Routes are global within GCP
    - GCP looks for subnet routes first, if none are matched it then looks for custom routes.
    - If using peered networks, GCP checks its local network routing table first, and then looks at peered network routes
    - Lower number = higher priority
    - System Generated Routes 
        - Added whenever a primary or secondary subnet is added to a VPC
        - When networks are peered, routes are added to the routing table
        - 2 types, subnet routes and default routes
        - Default route is for a route out of the VPC & towards the public internet. It also provides access to GCP API's
        - You can delete the default route if required
    - Custom Generated Routes
        - Static or dynamic
        - Dynamic is for cloud routers only
        - Static routes are applied to an instance tag OR the whole subnet
    
### Firewall Rules
    - Firewall rules are enforced at instance level
    - Egress is allowed by default
    - Allow & Deny rules are NOT available for implied rules
    - Lower priority number is preferred
    - Deny rules are processed before allow rules IF they're the same priority
    - Rules with same priority and action have the same result. The rule that is actually used is indeterminate
    - GCP only supports TCP, UDP, ICMP & IPIP protocols/ports

### VPC Peering
    - Allows private communication between VPC networks without the need for external IP's
    - Each VPC has to accept a connection in order to establish a peering
    - VPC's that peer cannot have overlapping subnets
    - VPC subnet routes are shared between VPC's. Custom routes are NOT shared by default
    - Project owner, project admin and Network admin IAM roles are required to configure a VPC peering
    - Firewall rules are not shared between VPC networks
    - Transitive peering is NOT supported

### Shared VPC
    - Allows an organisation to connect resources from multiple projects to a common VPC network
    - Create a host project to assign multiple service projects too
    - The VPC's within the host project are known as shared VPC's
    - Shared VPC admin can assign roles to service project admins for access to the shared VPC
    - Interconnects, VPN's & VPC peering should ideally be configured to the host project, so service projects can access those resources
    - Load balancing should be created at the service project level
    - A project cannot be both a host project and service project
    - You can only connect a service project to one host project at a time
    - IAM roles are needed to allow service projects to access shared VPC network resources
    - roles/resourcemanager.projectIamAdmin & roles/compute.xpnAdmin are the required roles to create a Shared VPC

### Cloud NAT
    - Allows internal compute devices & GKE clusters to access the internet through a NAT
    - Cloud NAT is region specific
    - GCP requires a cloud router to be configured in order to use Cloud NAT
    - Cloud NAT uses default route to access internet
    - Cloud NAT does not allow incoming NAT connections to the VPC
    - Even if enabled for a subnet, if a VM is spun up with an external IP it does NOT use Cloud NAT to access the internet
    - GCP allocates 64k ports per VM instance to an external IP for translation

### Private Google Access
    - Allows you to connect to GCP services internally without needing an external IP address
    - If Private Google Access is enabled for the subnet, VM's within that subnet (that only have internal IP's) can access the GCP API's via the default route of the VPC routing table
    - Egress firewall rules must also be enabled to allow the traffic from the subnet to the GCP services - apply this target tag to VM instances that require access


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


## Load Balancing

### Overview
    - Layer 4 Load Balancing
        - Ability to direct traffic based on data from network layer such as IP or protocol
    - Layer 7 Load Balancing
        - Allows for decisions to be based on application layer data and attributes such as HTTP header, uniform resource, identifier etc
    - Global Server Load Balancing
        - Extends L4 and L7 capabilities so they are applicable across geographically distributed server farms
    - Forwarding rule specifies IP address, protocol and port to be forwarded to a proxy
    - If a region is working at full capacity of load, new requests are forwarded to another region
    - Load balancing modes can be based on connections, CPU utilisation, or requests per second rate

### Global HTTP/HTTPS Load Balancer
    - Destination ports include 80, 8080 for HTTP and 443 for HTTPS
    - HTTPS load balancer uses a regional external IP address which must be IPv4 and use a regional external forwarding rule
    - In standard tier, it can only forward traffic in a specific region
    - Forwarding rule directs traffic to the internal HTTP/HTTPS proxy
    - Forwarding rule also provides an IP address which can be used in DNS records - no DNS load balancing is required
    - Target HTTP/HTTPS proxy terminates external client requests and opens new requests to backend instances on the internal network. It also checks the URL to determine which server is best suited for that request
    - HTTPS proxy requires an SSL certificate installed on the HTTPS proxy
    - URL map uses layer 7 URI components/rules to make intelligent routing decisions. For example, traffic with /video in the URL will direct traffic to backend service A whereas URLs without can direct traffic to backend service B
    - Backend service directs each request to an appropriate backend based on zone, instance health and server capacity of its attached backends
    - If you're using HTTPS or HTTP2, SSL certificates must also be installed on the backend instances
    - If a server needs to be removed from a pool, connection draining can be enabled on the backend service to ensure minimal interruption to the users
    - Session affinity means sessions from 1 client always go to the same backend instance. There are 2 types - by client IP or by setting a cookie on the client
    - Storage buckets can be used (instead of servers) by backend services to host static content sites
    - Firewall rules must be added in order to allow traffic from the load balancer to the backend instances. These are the IPs that allow connectivity from the load balancer to the backend instances
    - The firewall rules are enforced at instance level, not at traffic coming in towards the load balancer

### TCP & SSL Proxy Load Balancing
    - SSL Proxy Load Balancer uses TCP with SSL offloading
    - TCP Proxy Load Balancer uses TCP without SSL offloading
    - A client initiates a connection with the proxy load balancer, this connection remains open whilst the load balancer opens a second connection to the backend instances
    - When using premium tier networking, you are able to reserve an Anycast IP address for load balancer
    - Anycast IP address is an external IP address which can enter the GCP network via any edge POP, typically closest to the user
    - Standard tier networking the traffic is routed to the region where the backend instances reside
    - Traffic from the target proxy to the VM instance is over either TCP or SSL (Google recommendation)
    - Offers client session affinity
    - Health checks are performed on the backend service to the VM instances
    - Front end SSL certificates can either be managed by you, or GCP managed. SSL certificates between the backend services & VM instances are Google managed only
    - Any vulnerabilities which apply at SSL or TCP stack, Google will apply patches automatically

### Network Load Balancer
    - Regional, non-proxy load balancer
    - Allows client connections to pass through to backend instances
    - Target pools are pools of MIG
    - Target instances are single VM instances 
    - External traffic which hits the load balancing IP is then forwarded on to the target pool or target instance via the IPv4 forwarding rule
    - Existing connections are forwarded to the same server based on a hash, source IP & destination IP from the client
    - Return traffic goes directly back to the user and not back through the load balancer - known as Direct Server Return
    - Target pools are used for session affinity
    - Health checks are done on HTTP port 80 - even if server isn't running HTTP is must enable the server for the health check to be performed
    - Firewall rules must allow inbound traffic from the client source IP's as the load balancer is pass through
    - If the service is required to be accessible from the internet, GCP recommends creating an 0.0.0.0/0 firewall rule
    - Each forwarding rule can each have a regional external IP address. A use case could be different IP's for different services running on different VM's

### Internal TCP/UDP Load Balancing
    - Use case for this is web frontend servers which (if needed) can then forward traffic onto internal servers via an internal load balancer
    - The internal load balancer's IP is taken from the VPC primary subnet range in the region we select to create the forwarding rule
    - As long as the VM instances are configured in the same region, this internal load balancer IP is accessible from Cloud VPN, Cloud Interconnect or VPC Peering if configured
    - Internal forwarding rules must be in the same subnet as the internal IP address assigned to the forwarding rule
    - The subnet must be in the same region as the backend services that are connecting to it
    - You can have multiple forwarding rules connecting to the same backend services but each rule must have its own internal IP
    - GCP sends a health check probe to the IP address of the internal TCP/UDP load balancer
    - GCP does not have a UDP health check
    - The VM instances must be configured with the same IP address as the load balancer
    - Session affinity is done on a best effort basis. UDP does not support session affinity
    - Forwarding rules can have up to 5 different ports configured on them to forward traffic
    - Forwarding rules cannot be modified and must be destroyed and rebuilt when changing them

### Managed Instance Groups
    - Contains identical instances as a single identity in a single zone
    - Zonal instance groups & regional instance groups
    - Regional instance groups improve availability by spreading instances across multiple zones in a single region
    - Unmanaged instance groups are a group of devices that do not share a common instance template
    - Unmanaged instance groups do not create, delete or scale the amount of instances in the group
    - Autoscaling is a feature of managed instance groups
    - Autoscaling is defined by an autoscale policy and is used to determine when to scale a group
    - By default, 60 second delay for the autoscaling policy to obtain info about the new instances - this can be changed
    - There is a 10 minute stabilisation period for scale down activities. The autoscaler policy monitors load for that 10 minutes before scaling down

### Network Endpoint Groups
    - Backend for GCP load balancers
    - Allow us to create logical groupings of IP addresses and ports representing software services instead of VM's
    - VM must be in the same zone as the endpoint
    - Do some more research on this??????

### Cloud Armour Policies
    - Limited to global HTTP/HTTPS load balancer. It does not support internal HTTP/HTTPS load balancing
    - Security polices that we define to allow or deny traffic at the perimeter of GCP, stopping traffic from entering our VPC network
    - To enforce this you can create an allow list which works alongside a default deny
    - Alternatively you can use a deny list with a default allow rule
    - Cloud Armour can mitigate DDOS attacks by blocking IP's trying to attack the GCP network
    - Only one Cloud Armour policy is allowed per backend service
    - To create a policy, you need to be assigned the security admin role
    - Setting a policy for a backend service, you need to be assigned the network admin role
    - You can view traffic logs in Stackdriver
    - You can put up to 5 IP addresses or ranges per rule
    - Preview mode allows you to see the rule in action without it being enforced, so you as the engineer know whether the rule should be enforced or not


## DNS & CDN Network Services

### Cloud DNS & CDN
    - 