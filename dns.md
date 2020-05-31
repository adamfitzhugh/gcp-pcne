## DNS & CDN Network Services

### Cloud DNS & CDN
    - Types of DNS Zones in GCP
        - Internal DNS - provides host name resolution for all compute instances within a VPC. This is different to Cloud DNS
        - Private Zone - contains DNS resorces that are only visible internall within your GCP networks
        - Public Zone - is visible to the internet. Usually purchased through a registrar

### Managed DNS Zones
    - Private Zone
        - By default in a private zone, both NS and SOA entries are created
        - You create your private zone and attach the names of the servers to that zone by their IP addresses as A records. This allows internal DNS lookups to this URL from the VPC networks you choose
    - Public Zone
        - Next, create a public zone for the URL
        - Update name servers to forward queries to Cloud DNS name servers
        - Create A and CNAME records for access to the servers
    - Migrating Public Zones
        - In GCP, create public zone
        - Export DNS records in either BIND or YAML format from your on-premises network
        - Import DNS records into GCP from on-premises
        - Head to registrar to update name servers to Cloud DNS name servers

### DNS Forwarding & DNS Peering
    - DNS forwarding is used to forward DNS queries from a GCP VPC network into an on-premises network, where the physical server is located. This works by creating a GCP DNS private zone, selecting the VPC network where the host server is, and ensuring the on-premises destination server IP address is configured
    - You cannot DNS forward between 2 GCP projects
    - DNS resolution order on GCP
        - DNS server policy rule
        - Private DNS forwarding zones
        - Private and peer zones
        - Compute engine internal DNS
        - Queries public zones
    - DNS Server Policies are used to route inbound or outbound queries to/from GCP VPC network
    - DNS Peering
        - Allows us to connect 2 different VPC network to share DNS resolution services
        - Producer Network - produces the DNS 
        - Consumer Network - uses the producer network for DNS queries

### DNS Security
    - Protects public domain zones from spoofing
    - Create new public zone and turn DNSSEC to on - before switching this on you need to check with your registrar that DNSSEC is supported
    - Configure client DNS resolver that validates signatures for DNSSEC or use public resolver
    - Add DS record through your domain registrar to activate DNSSEC
    - Transfer state is used when youre migrating domains to or from GCP DNS
    - When migrating public zones to GCP with DNSSEC on you migrate as described above but in step 1 select the DNSSEC state as transfer. You also need to export and import the DNSKEYs. Then finally, turn DNSSEC from transfer to on

### Content Delivery Network
    - Stores a cached version of content in multiple geographic locations to improve performance
    - Requirements
        - GCP Premium network tier
        - Global HTTP load balancer
        - Edge Location cache server
    - CDN supports images, video, audio and other file types (such as CSS, JSON etc)
    - Cache Miss is when a users request comes into CDN and it doesn't have the content. If this is the case the CDN will request the data from the region where the data resides
    - Cache Fill is when the data is sent from its location into CDN

### Cloud CDN Cache Controls
    - Each cache entry in a CDN cache is identified by a cache key
    - When a request comes into cache the cache converts the URL request into a cache key and checks to see if they key already exists in cache
    - You can create custom cache keys
    - Cache invalidation is used in order to remove cached paths which are no longer required. This is not immediate
    - Make sure you set proper cache expiration time for time-sensitive content such as sports scores

### CDN with Signed URLs
    - gcloud compute sign-url is used to create CDN key for signed URLs
    - Enables you to service authorised URL requests
    - Signed URL is given an IAM role in order to allow access to backend services
    - Must configure max age for the signed URL