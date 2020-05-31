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