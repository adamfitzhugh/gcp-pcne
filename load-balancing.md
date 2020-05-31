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