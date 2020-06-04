## Hybrid Networks

### Hybrid Connections
    - VPN, Interconnects & Cloud Routers
    - LOA-CFA is sent by Google to us on-premises which we forward onto our colocation facility to provision cross connects between GCP and us on-premises
    - Dedicated Interconnect - where our colocation facility provider provisions a circuit between our network and GCP
    - Partner Interconnect - used if we do not want minimum 10Gbps speed of a dedicated interconnect OR if our on-premises DC cannot reach a dedicated interconnect colocation facility
    - VPN securely connects our network to GCP network over IPSec VPN tunnel
    - Direct Peering is used to connect on-premises network directly to GCP Edge Network
    - Carrier Peering works exactly the same way as direct peering except we use a shared network from our service provider to access GCP. We would use carrier peering if we didn't need full 10Gbps speeds to GCP

### Dedicated Interconnect
    - Provides physical connection between our on-premises network and our GCP VPC network (direct to VPC)
    - Supports only dynamic routing
    - Supports minimum 10Gbps & up to 200Gbps
    - Select a metropolitan area closest to our on-premises network and establish a connection to GCP
    - On the LOA-CFA form, we specify what speeds/connections we need. We can ask for up to 8 10Gbps connections or 2 100Gbps connections
    - Recommended to create secondary interconnect link for redundancy purposes
    - Once setup we create a BGP peering between on-premises network and the cloud router within our VPC
    - You can add vLAN attachments to already existing dedicated interconnects
    - You can choose different bandwidths per vLAN attachment across the interconnect. Maximum speed is 50Gbps

### Partner Interconnect
    - Partner interconnects are used if our on-premises DC cannot reach a dedicated interconnect colocation facility OR if we do not need 10Gbps
    - Most connections are layer 3, but some offer layer 2 connections
    - When we create a vLAN attachment for a partner interconnect, GCP creates a pairing key that we provide to our service provider in order to connect the service provider connection to our VPC network. The key can only be used once!
    - No need for an LOA-CFA
    - The region you select for the vLAN attachment must be supported by a partner interconnect location which is a city where the service providers network meets Googles network
    - Connection capacity can range from 50Mbps to 10Gbps

### Cloud VPN
    - Securely extends GCP network through IPSec VPN tunnel
    - 2 VPN options - HA VPN & Classic VPN
    - Routing options include BGP, Route based or Policy based
    - You can advertise Cloud router advertisements or select custom routes to advertise
    - In HA VPN dynamic mode, a cloud router is mandatory
    - HA VPN interfaces are created regionally
    - Cloud VPN supports 3Gbps maximum transfer rate per VPN tunnel & an MTU of 1460. MTU can be adjusted
    - Cloud VPN supports IKEv1 & IKEv2
    - Routing traffic between 2 on-premises sites through GCP (like a transit) is not allowed
    - If the MED for 2 VPN tunnels is the same, GCP will use ECMP to balance traffic across both links
    - GCP recommends active/passive routing on HA VPN, because if a link fails on an active/active set up, it can cause some issues in terms of available bandwidth
    - Classic VPN offers 99.9% uptime

### vLAN Attachments
    - You can limit bandwidth for specific vLAN attachments. If there is available capacity, the limit is not enforced
    - With partner interconnect, our service provider provides a portion of a shared connection to GCP using layer 2 or layer 3 connectivity
    - Pairing key is given to our service provider in order for them to connect their network equipment with our VPC network
    - If there's maintenance in a colocation zone 1, that will be across all zone 1's. This is why it's important to have connections in multiple zones in the colocation facilities

### Cloud Router
    - Dynamically exchange routes between on-premises and GCP networks
    - Uses BGP to exchange routes
    - Subnets between on-premises and GCP cannot overlap
    - Regional Mode - cloud router only shares the subnets that were created within that region
    - Global Mode - cloud router lets you dynamically learn routes to and from all regions

### Peered Connections
    - Useful for cutting egress fees from GCP
    - No setup or maintenance costs to set this up
    - Reduce latency and increased availability
    - Direct Peering, Carrier Peering & Private Peering
    - Direct Peering requirements
        - Publically routable ASN
        - Publically routable address space
        - ASN record in peering DB

### Highly Available Connections
    - Google recommends 4 9's of availability
    - Billing credits are issued to the customer if the target availability times are not met
    - 99.99% design requires 4 cloud routers, 4 interconnects, 4 interconnects to colocation facilities
    - MED is used to distinguish the same advertised networks
    - MED values are set on the on-premises connection. If none are set, Cloud Router assumes the highest possible MED value of 0
    - A single cloud router does support multiple connected interfaces and multiple endpoints. However, each cloud router must use the same ASN number for all of its BGP sessions EXCEPT for partner interconnect
    - ECMP is used when the connections have the same MED value
    - You cannot connect a single cloud router to multiple ASN numbers defined on-premises. If this happens, the cloud router will only accept routes from the lowest ASN number
    - MTU is 1460 over VPN tunnels