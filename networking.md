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