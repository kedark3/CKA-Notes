# Basics


## OSI model 7 Layers

OSI model has following 7 layers(pnemonics in the paranthesis):

- Application(away) - How applications access networks, like web browing, file transfer, ssh
- Presentation(pizza) - Data conversion and formatting. Encryption, file formats are managed here
- Session(sausage) - Track network communication
- Transport(throw) - Breaks down data in `Segments`(in case of TCP)/`Datagrams`(in case of UDP) adds Header with Sending and Receiving port numbers to the data.
- Network(not) - uses IP protocol, adds header with source and destination IPs and the data is now called a `Packet` at this layer
- Data Link(do) - Goal is to get data from one device to another, within network. This layer does not try to get data to final destination, that information is added by layers above in the headers. This layer simply tries to forward packets to next device where it thinks the packet will find route to destination. Adds header with Source and Destination MACs. Trailer may be also added with error correction infomation. now the data is called `frame`. Destination L2 address, i.e. MAC address in the header of a frame keeps chaning with the each hop, when one device gets it and thinks it needs to send the frame to another device. It updates the destination MAC and forwards it.
- Physical(please) - This is actually sending 1s and 0s or electrical or wireless signals.

## TCP/IP Model

Originally developed by US Dept of Defence and then got more adapted by universities and got popular. Original model had 4 layers, but current version is 5.

- Application(combines Application+Presentation+Session) - HTTP, SMTP, FTP, IMAP
- Transport - TCP/UDP
- Network - IP, ICMP, ARP
- Data Link - Ethernet 
- Physical - Physical Transmission.


### TCP

- Not lightweight as it has more features
- Connection oriented
- Data called as Segments
- Reliable, has error recovery
- Windowing(flow control)
- Ordered Data transfer using sequence numbers
- Uses Source and Dest ports, ports enable multiplexing, so many applications can use the network at same time. If we receive two different network communications, we can't figure out which process owns it until we know what Port it is going to. Because IP and MAC addresses are same in both the case.


Three Way Handshakes:
- Client sends Empty TCP header with SYN, uses SYN flag for synchronous conversation, Window value is also set.
- Server sends Empty header with SYN/ACK- Says I acknowldge you and let's have synchronous communication
- Client sends Empty TCP header with ACK flag set.
Random sequence number is used in all 3 packets above. First is selected random and next two will be incrementing that random sequence number.

Graceful Termination:
- Dev1 sends FIN/ACK, Dev2 says ACK
- Dev2 Sends FIN/ACK, and Dev1 says ACK
These are required because when initial FIN/ACK is received, the application may need little time to send FIN/ACK of its own.

Abrupt Termination:
- RST(Reset) message is used to close connection if there is error in the connection

### UDP

- Lightweight
- Connection less
- Data called as Datagrams
- Does not care about data sequencing
- Unreliable, doesn't care about missing packets or errors(useful in audio/video data)
- Uses Source and Dest ports, ports enable multiplexing, so many applications can use the network at same time. If we receive two different network communications, we can't figure out which process owns it until we know what Port it is going to. Because IP and MAC addresses are same in both the case.



## IP Addresses

- Network Layer
- dotted decimal notations
- 32 bit long
- 4 octets of 8 bits
- range between 0-255
- IP addresses belong to the network and not to the device
- IP can be static address or assigned dynamically with DynamicHostConfigurationProtocol(DHCP)
- Under IP layer, packet is referred as "IP Datagram"
    - Maximum size of datagram is 65,535 (data that you can represent with 16-bit)
    - IP datagram has TTL field - set to 64, every time it completes one hop, between one network device to another, it is decremented and if it reaches 0, then datagram is no longer forwarded.
    - Protocol field mentions if it is TCP or UDP protocol

IP address Classes:

- Class A: 1st octet is network ID, last 3 for host ID. Can contain 16 million addresses.
E.g.

| 9   |      100.100.100      |
|----------|:-------------:|
|NetworkID |    Host ID|




- Class B: 1st two octets are network ID and last 2 for host ID. 64000 addresses.
- Class C: 1st three octets are network ID and last for host ID. 254 addresses only.

Non Routable address:
- Following three addresses are non-routable via public/core routers and can be used freely within a private network.
    - 10.0.0.0/8
    - 172.16.0.0/12
    - 192.168.0.0/16
- Exterior Gateway Protocols (EGP) won't route those


## DHCP

- New devices in the network ask for IP from DHCP server.
- New devices send DHCPDISCOVER message as Broadcast(as it doesn't know where the DHCP server is) and asks for an IP.
- DHCP server will receive this message while other devices in the network will ignore it.
- DHCP server looks at the address pool available and reserves an IP and sends back to the requesting device as DHCPOFFER.
- If multiple Offers are received, device will accept one of the OFFER and send back DHCPREQUEST to use that address to dhcp server.
- Lastly DHCP server sends Unicast message to Client with DHCPACK to allow using that address.
- DHCP addresses are leased for a certain time and when the lease is up, the IP address is claimed back by DHCP server. The clients can send RENEW lease message to server to continue to hold on to the lease.
- Client may send RELEASE message to release the IP it was leasing.

## ARP - Address Resolution Protocol

- Used to find HW address of a node with certain IP address
- All devices in network maintain ARP table - it contains MAC and IP address mapping for all devices known to it
- Used when creating Ethernet frame to transmit to some IP address.
- When sending computer doesn't know MAC address of receiving computer, it uses ARP.
- Working:
    - Suppose you are sending data to 10.20.30.40, your computer doesn't have that address in the ARP table
    - It sends the ARP broadcast on the network, as `FF:FF:FF:FF:FF:FF` and its received by all systems in the network
    - Receiving systems check if the IP address in this message matches to their own IP address, if so it sends back ARP response - Unicast to node who sent ARP request
    - Now the sender knows the MAC address of recipient, Ethernet frame is filled with that MAC address and it is sent to the network.
    - Sender also updates its own ARP table with the entry so if it needs to send to same IP again, it can skip the ARP discovery.
- ARP entries expire after a short time to ensure changes in the network are accounted for.

## Subnetting

- Split large networks into smaller ones.
- Subnet masks are 32 bit long. Divided into 4 octets like IP addr.
- E.g.


For network 9.100.100.100:
| 255      | 255      | 255      | 224      |
|----------|----------|----------|----------|
| 11111111 | 11111111 | 11111111 | 11100000 |

Shorthand way to represent this is: 9.100.100.100/27 
- Subnet masks is a simple way for computer to use **and operator** to tell if an IP is on the same network or different.

## CIDR - Classless Inter-domain Routing

- Describes blocks of IP addresses.
- Demarcation point - To describe where one network or system ends and another one begins
- CIDR Notation `/xx` suffix for a network.
- Helps us to not care about network class, and use CIDR to determine what a particular network means.
- Allows more flexibility, than class A/B/C, allows you to have NETMASK of /23 or /25 or anything else.

## Hubs and Bridges

- Hubs are layer 1 devices. As they just operate on physical layer of the network. They have no intelligence. Its just a way to connect bunch of devices to one another through a central device. They are alos Collision domains as traffic is usually sent to all devices and only relevant device would accept it while others discard it. This is not very useful as it can be a security risk and unwanted devices may see your data. Hubs also only allows one device to communicate at a time and if collision happens, devices wait random time interval and try again. They could also use Carrier Sense Multiple Access(CSMA) protocol to see if the link is busy and if not then only send the data. CSMA has two submethods - Collision Sensing or Collision Avoidance. Hubs are old and obsolete now. Internally it was using BUS topology.
- Bridges was another way to make networks better. Bridge would segment larger network segments into smaller and connect all those together. Now if the traffic is sent from one MAC to other MAC, a Bridge would try to create a dynamic table in its own memory to track which MAC address lives on which part of the network. So if it receives traffic on interface 1 it knows the sender lives on interface 1 and marks it in the table. This way every time traffic is passing through bridge, it learns which of its interfaces have access to what devices and it will also add Auto-expiry timer on each entry so stale entries can be removed if device is down or missing. Bridge also reduces collisions, as it only forwards the traffic across the bridge if it belives the destination mac is on another interface than the source mac it received traffic from. Hence the Bridge became Layer 2 device. It works at MAC address layer by being intelligently managing traffic at Data Link layer using MACs. We use this less now in favor of the Switches.Internally it was using BUS topology too.

## Switching

Let's assume that we have two VMs(or cloud instances) that are trying to communicate with each other in the same network. We connect both of them to a switch and then assign them an ip address in the same network. Use `ip addr add`. Once IPs are assigned and interfaces are UP the computers can talk to one another. The switch can only allow communication within a network.

- Switches are the devices we use today and it is better than Hubs/Bridges. It uses Star Topology internally.
- Switch can use:
    - `Store and forward` method for data transfer. where it waits for entire data frame to arrive, check for errors and then decide to forward it. It is a safe method.
    - `Cut through` method waits only long enough to see destination mac address, and once the dst mac is known, then it starts forwarding the data bits immediately. No error checking at the switch. Its a fastest method.
    - `Fragment free` method is compromise between above two. It stores first 64-bits of the frame it receives and checks errors, if there are no errors, now everything is forwarded immediately.
- Above 3 methods are usually set by manufacturer and you don't control it. But you can research these before buying a new switch.
- Switches could be unmanaged, managed at Layer2 or managed at layer3.  In a layer two switch, there is not an ARP table, only a forwarding table. The switch records each src MAC address it sees inbound in the forwarding table, and attributes it to the port so frames with a dst MAC will only get sent to the port known for that MAC. Many people call this an "arp table" or "arp cache" even though it is neither.

- In a managed layer two switch, there is a forwarding table plus an ARP table but the latter is only used for the management interface to talk to interested hosts (i.e. the PC you are using to configure the switch.) In a managed layer 3 switch there will be a forwarding table plus an ARP table, since it needs it for the management interface plus router functionality exists to perform forwarding between subnets.



## Routing

- A network device that forwards traffic depending on the destination address of that traffic
- Works in 4 basic steps:
    - Receive Data packet
    - Examine destination IP
    - Look up IP destination network in routing table
    - Forwards traffic to destination

E.g.

Now let's assume there are two different networks with few devices earch. Now if the device A in network 1 wants to talk to device B in network 2 then swith is not sufficient and that's where a `Router` comes in. It is an intelligent device. Since in this simplified example it is connected to 2 networks, it gets assigned an IP in both the networks. So for example, in the network `192.168.1.0` it will have IP `192.168.1.1` and for `192.168.2.0` network, it will have IP `192.168.2.1`.

Since the router is like just another device on the network, we configure systems on the network with a gateway address, where it learns what address to reach out to if the destination device is not on the same network. And this gateway is usually an address of the router.

So route would look like `192.168.2.0/24 via 192.168.1.1` so telling device that I can reach the network via `192.168.1.1` route.

But when it comes to different networks that are outside of your network, you really can't set route for each entry. Hence you just add a `default` gateway, so it will look like:
`ip route add default via 192.168.1.1` or `ip route add 0.0.0.0 via 192.168.1.1` telling our system that any unknown address should be routed through router and you will find the destination.

Although, If you have two different routers in your network, one for private and another is for public routes, you need to have routes for both of the routers as gateways.

- **Routing Table** 
    - contains list of where to send traffic depending on what destination address.
    - It usually have a catch all entry for any destination addresses it doesn't know.
    - Although these tables can vary wildly based on make and model of the router, there are a few common fields in most routing tables.
    - Destination Network: Contain a row for each network the router knows about. Stored as x.x.x.x/yy format (CIDR notation)
    - Next Hop: This is the IP of next router, which should receive the data based on the destination network
    - Total Hops: how many hops ot will need to get to destination address.



## VLANs

- Helps you divide physical LANs into smaller VirtualLAN(VLAN)s without needing extra Switches, Routers etc.
- VLAN can be created within a single Switch as that's economical and scaleable.
- Adds security to the large network by separateing the traffic.
- Separate traffic for various purposes. Like Data/Voice/Management VLANs.
- Limits Broadcasts and Flooding
- Smaller Failure Domains
- Each VALN has an ID, between 0-4095, but 0 and 4095 is reserved, so you can use 1-4094
- Typically you have `One Subnet per VLAN` as a best practice.
- VLAN is a switching technology and they live at Layer 2. Talking across 2 different VLANs can be done via Routers.
- Switches usually always have VLAN 1 set on all the ports by default.


## Trunking

- If we have multiple switches and want to interconnect them, and we also have number of VLANs. We may be able to physically connect one port on each switch to assign it to a particular VLAN. But that way we will exhaust our ports quickly and won't have enough space for actual physical devices to connect to the switch. That's where we have `Trunking`.
- In Trunking(or Tagging), We can use single link between switches, and have many VLANs connected on a single port. An `Access port` is how the devices access the network. A `Trunk Port` is like a single tree trunk with many branches (or VLANs)
- On Trunk port, traffic is identified using the `VLAN ID` in the Ethernet frame.
- Broadcast or Flooding is possible across multiple switches as VLAN enables limiting traffic to ports which are assigned to intended VLANs. And we will travel over trunk ports to other switches.

- Trunk port == Tagged port and Access Port == Untagged port.
- Tagging uses 802.1q standard from IEEE.

## Native VLAN

- It was created to support devices like hubs that didn't understand VLANs. So if they can't tag the traffic with VLAN, then what VLAN would the traffic belong to? Answer, `Native VLAN`.
- When untagged traffic is sent from a device, it can be assumed to be native VLAN.

### IGP - Interior Gateway Protocol

- Distance vector protocol - older method of doing it and in this router, sends data about list of all networks and how far they are in terms of distance(hops) to other routers. They have no idea about what is the actual state of the links/network.
- Link-state routing protocol
    - Each router advertises state of its link of each of its interfaces
    - Infromation about each router is propagated to all others in the network

- Routers within same organization use this protocol

### EGP - Exterior Gateway Protocol

- If the routers are sharing the information across organizational boundries, with other routers owned by different entity, they use this protocol.
- These are the routers at the Edge of the networks.

## DNS

To use names for systems instead of their IP addresses, as names are more easier for humans to remember, we need to use DNS.

Before DNS, you would put info about the host in `/etc/hosts` file as:

```sh
cat /etc/hosts
192.168.1.19 DB
```
Now this way when we `ping DB` it pings the addr `192.168.1.19`.

You could even fool your system by changing it as :
```sh
cat /etc/hosts
192.168.1.19 www.google.com
```

Now if you ping google, you get response from `192.168.1.19` which is not actually google, its just some system you set address to.

This process of resolving name to an IP address is known as Name resolution. And pre-DNS people had to maintain `/etc/hosts` files on all systems to enable name resolution. If a system IP was changed, the updates would need to be in each of the system that wants to talk to this system. In big network, its impossible to maintain such files.

Hence we created a single central server called DNS server. And we point all our servers in the network to point to contact DNS server to look up names.

It is set in:
```sh
/etc/resolv.conf
search lab.example.com
nameserver 192.168.1.100
nameserver 8.8.8.8
```

This way our system would look at Nameserver entries one by one to try to find the name its looking for.

So in the above example, we may have private DNS server within our company at `192.168.1.100` and we all know that google hosts a popular public DNS server at `8.8.8.8` so any website/hostname we try to access, we will first check it in our private DNS server and if not found, we will try the public DNS server. The default behavior for resolv.conf and the resolver is to try the servers in the order listed. The resolver will only try the next nameserver if the first nameserver times out. You can alter the resolver's behavior using rotate, which will query the Nameservers in a round-robin order.This has the effect of spreading the query load among all listed servers, rather than having all clients try the first listed server first every time.However, nslookup will use the second nameserver if it receives a SERVFAIL from the first nameserver. From the nslookup manpage:

    [no]fail Try the next nameserver if a nameserver responds with SERVFAIL or a referral (nofail) or terminate query (fail) on such a response.

    (Default = nofail)

Also, if you look at the `search` entry, it is used to short circuit name lookups. For example, if I have a server named `db.lab.example.com` and I just say `ping db` instead of its `fqdn` then the `search lab.example.com` will be invoked to complete `db` to `db.lab.example.com` and search that on the name server. This is useful to have in private networks so you don't have to always type out the entire name.

Order of lookup between local files or DNS servers, meaning where to look first, on local system files or DNS servers, is configured by `/etc/nsswitch.conf`

Names are divided into top level domain, domain, and subdomain. E.g. in `www.google.com` or `maps.google.com`, `maps` and `www` are subdomain, `google` is domain and `.com` is top level domain name. And at the root the domain is just `.` ->`.com` ->`google`->`all other subdomains` forms a tree structure.


# Making a linux host act as a router

Let's say now we have 3 devices, A,B,C. If A is connected to B, B is connected to A&C and C is connected to B. If A wants to talk to C or C to A, we need route on each of A and C as hosts(otherwise ping would get network unreachable error from either A or C):
```sh

# on A
ip route add 192.168.2.0/24 via 192.168.1.6 # which is the IP of B on network 192.168.1.0
# on B

ip route add 192.168.1.0/24 via 192.168.2.6 # which is the IP of B on network 192.168.2.0

# B basically would have two IPs on each of the network
```

Now if we ping, we don't get error. But we will remain stuck with no output. No response back.

This is because, there is no packet forwarding on Linux by default for security. This is because if eth0 is connected to public nw and eth1 to private, we don't want anyone to send messages from public network to private network and exploit it.

We need to enable it using kernel tunable if you are aware of all the networks being safe and private.

`cat /proc/sys/net/ipv4/ip_forward` and then set this to `1` using sysctl or in `/etc/sysctl.conf` file.


**NOTE:** Any changes made using ip addr command do not persist across reboot and you need to update network configs actually to take it effect.

### Record Types
- A Record - Hostname and IPv4
- AAAA Record - Hostname and IPv6
- CNAME Record - A Record name and its aliases. Example: `food.web-server` is A record and can be aliased as `eat.web-server` and `hungry.web-server`.

### Tools to check DNS

- NSLookup- Queries DNSserver, doesn't honor entries in `/etc/hosts` file
- Dig - Looks up DNS server and returns info as it may be stored on DNS server with A Record Cnames etc.

# Container Networking From Scratch

## Network Namespace and virtual switches/bridges

- In the description below, network namespaces are synonimous to containers.

- Containers usually have their own Routing tables, ARP tables, and interfaces. And they differ from underlying hosts Routing and ARP tables.
- To create or manipulate network namespaces, you should use `ip netns` command.
- `ip netns exec <nsname> <command>` will allow you to execute commands within a namespace.

- Just like physical world, we are creating a virtual network using `ip netns` commands. Now for multiple namespaces to interconnect with each other, we would need a virtual Switch.E.g. Linux Bridge, OVS(Open vSwitch)

- Using `ip link add type veth peer name <some_name>` typed links, we can create virtual cables/links to connect the namespaces with one another. And connect the namespaces to one another directly with this virtual link.

- But with the intorduction to vswitch(when we have more than 2 namespaces to connect), we need to connect all links to vswitch and all namespaces talk to one another through this vswitch.

- A linux bridge can be added as `ip link add v-net-0 type bridge` where `v-net-0` is name of the virtual linux brige.

- A Virtual bridge only allows communication between namespaces, but can't directly communicate outside of the host. In such situation, NAT(network address translation) can be used, with MASQUERADE method, where, any traffic coming from virtual bridge network will be replaced with IP address of the host before it leaves the host, hence, other hosts see the traffic is coming out of the host and not from the namespaces within the source host. And they will respond back to the host.


## Docker networking

- When installed on a host, Docker creates a virtual bridge on the host similar to what we have seen earlier in namespaces.
e.g.
```sh
docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:71:01:5b:e4 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:71ff:fe01:5be4/64 scope link 
       valid_lft forever preferred_lft forever
```
- When you allow docker to port map with host port, e.g. `-p 8080:80` would map host port `8080` to container port `80` you can see it in `iptables` as follows, as an example:
```
# netstat -tunlp | grep 8080
tcp6       0      0 :::8080                 :::*                    LISTEN      1254217/containers- 
```

### In Summary Network namespaces in the containers use following steps:

1) Create network namespace for each container
2) Create bridge network/interface
3) Create VETH pairs(Pipe, Virtual Links/Cables)
4) Attach vEth to Namespace
5) Attach other vEth to virtual bridge
6) Assign IP addresses
7) Bring the interfaces up
8) Enable NAT - IP Masquerade

These steps are also followed by Docker, RKT, k8s etc. with potentially some variations. This is what `CNI(Container networking interface)` came up with.

- Example Script to achieve the same:

```sh
# create veth pair
ip link add type veth peer name <some_name>

# Attach veth pair and bring up the link
ip netns exec <ns> ip link set veth-blue up

ip netns exec <ns> ip link set veth-red up

# Assign IP address

ip -n <namespace> addr add
ip -n <namespace> route add


# Bring up Interface
ip -n <namespace> link set
```
Although above script can help you establish networking, it is less ideal. A better way is to have a router in the network and all hosts point to router as the default gateway. The former part of script where container network namespaces are created and connected through bridges, this can't be done manually on the K8s. To do this automatically, CNI comes in acting as a middleman.

## Container Network Interface

It defines rules for container runtimes:
- Container Runtime must create network namespace
- Identify network the container must attach to
- Container Runtime to invoke Network Plugin(bridge) when container is ADDed
- Container Runtime to invoke Network Plugin(bridge) when container is DELeted
- JSON format of the Network Configuration

It also defines rules for Network Plugins:

- Must support command line arguments ADD/DEL/CHECK
- Must support parameters container id, network ns, etc.
- Must manage IP address assignment to PODs
- Must Return results in a specific format


NOTE: Docker does not implement CNI, but uses Container Network Model(CNM), hence doesn't natively integrate with CNI plugins like Weaveworks, Calico, Flannel.

- When K8s creates containers, it creates the containers, with None network, and then invokes the cni plugins to assign networks.

## Network architectures/models
* K8s Network needs to satisfy following requirements:
    - All containers can communicate to all other containers without NAT
    - All nodes can communicate with all containers and vice-versa without NAT
    - The IP that a container sees itself as is the same IP that others se it as

* Container networking basics:
    - Single Node single container:
        - usually in the networking context see a contianer as an isolated networking namespace
        - To connect the container with the default namespace on a host, assume you have a (virtual) ethernet cable, with two ends of it being called `veth1` and `veth2` where `veth1` is in `default` namesapce on the host and `veth2` is in the container namespace.
        - Routes from default namespace on host would say if any traffic comes on `172.16.0.1` route it to `veth1`, while on the other hand from container, default routing rule would be like `default via 172.16.0.1 veth2`
        - To achieve any namespace operations use `ip netns` commands.
    - Single Node two containers:
        - In this casem we use same model from above, but we need a bridge to allow communication between two containers.
    - Two nodes, multiple Containers, same L2 network:
        - Extend what we had in the previous step, but we have a switch connecting to nodes to one another.
        - You can use special routing rules on the each of the nodes so that when node receives a packet it know where to send it to. In our case, nodes will route it forward to the bridge and for that you may need to enable kernel tunable for ip forwarding(`net.ipv4.ip_forward=1`), as by default its disabled for security reasons.This kind of setup is possibly used by Flannel in one of its backends - `Node Gateway` implementation.
        - Notice as its L2, its all about routing table rules allowing the communication.
    - Two Nodes, Overlay network:
        - Previous networking model of L2 is limited, and only works if two nodes were on a same network and connected to a known switch.
        - In such case, you would create a tunnel device on both the nodes and this tunnel device would have a process(look at `socat` program) behind it.
        - Now when a packet is sent out from a container, it goes into bridge and from there, it goes into tunnel where tunnel process looks at what IP address its going to and using something like EtcD tunnel process able to send the package out to correct address of another node.
        - On the receiving end, the receiving tunnel would also do the same.
        - This process listed above is used by UDP Flannel CNI plugin backend.


## Flannel, Calico, Weave
- Flannel

  Has multiple backends
  - host-gw : Multiple nodes, same L2 network as we saw above
  - udp: what we saw in overlay UDP network
  - vxlan: same as above but implemented in the more efficient manner(more suited for production)
  - awsvpc: Sets routes in AWS
  - gce: Sets routes in GCE
  - Node-> pod subnet mapping is stored in EtcD.

- Calico

    - No overlay for intra L2, Uses next-hop routing(same as Multiple nodes Same L2 seen above)
    - Inter L2 communication uses IPIP overlay
    - Node->Pod-subnet mappings distributed to nodes using BGP.

- Weave
    - similar to Flannel uses vxlan overlay for connectivity
    - No need for etcd, mappings distributed to each node peer to peer.


# Cluster Networking Requirements

Within a k8s cluster, we have some basic networking requirements as below:

- Each node(master/worker) must have an interface connected to a network.
- Each interface must have an address, unique hostname and mac addresses.

Port that are required to be opened:

- Master should have port 6443 opened for kube-apiserver
- Kubelets should have port opened on 10250
- Kube-scheduler needs 10251
- Kube-controller-manager needs 10252
- NodePort service uses ports between 32000-32767
- Etcd server listens on 2379


# Pod Networking Concepts in K8s

K8s does not have a built in solution for networking.

- Every POD has an unique IP Address
- Every POD should be able to communicate with every other POD in the same node
- Every POD should be a ble to communicate with every other POD on other nodes without NAT
- There are Flaneel, Cilium, NSX, Calico plugins that are able to provide this networking in k8s.

# Networking APIs

* Service, Endpoints, EndpointSlice
    - for service registration and discovery
* Ingress 
    - L7 HTTP Routing
* Gatway
    - Nex-generation HTTP routing and service Ingress
* NetworkPolicy
    - Application "firewall"

# Networking Components

* Kubelet CNI(Container Network Interface) 
    - Low level network drivers and how they are used
* Kube-proxy
    - Implements Service API
* Controllers
    - Endpoints and EndpointSlice
    - Service load-balancers
    - IPAM
* DNS
    - Name-based discovery

# Networking Model

All Pods can reach all other Pods, across Nodes

There are many Implementations:

    - Flat
    - Overlays (e.g. VXLAN)
    - Routing Config(e.g. BGP)


# Services

## Problems it solves

One of the important problem services tries to solve is where servers(or pods) running the application:
    - have multiple replicas
    - and, any of the replicas may go down without notice
Clients accessing the applications over the network, may get confused if the pod with the IP they are communicating with, is lost. Hence, we need a Service.


## Solution is a K8s Service

Services "expose" a groupd of pods using:
    - Durable VIP(or not, if you choose)
    - Port and protocol
    - Used to build service discovery
    - Can include load balancing(but doesn't have to)

Also it is important to note a Service is a virtual entity.

## Services: What really happens? 

1) Client starts by doing a DNS service
2) DNS returns service VirtualIP(VIP)
3) Client connects to VIP
4) Kube Proxy intercepts the traffic and translates VIP to backed pod IP
5) Async: there are controllers running in the background

# Service Networking

In reality you rarely configure your pods to communicate directly with each other. If you want to access the services hosted on another pod you will use a service. If you use a NodePort service, all the nodes in the cluster expose the port required by the service, and accessible the any of the worker-node:port combination, service is still accessible and routes traffic to correct endpoints/backend pods. In the ClusterIP the service is only accessible within the cluster. It is important to understand that Service is a cluster wide concept and every service you create exists across all the nodes in the cluster. While it also doesn't exists at all as it is a Virtual object. When we create a service object in k8s, it is assigned an IP addr from a pre-defined range. 

The kube-proxy components running on each node, get's that IP addr and creates forwarding rules on each node in the cluster, saying any traffic coming to this IP, the IP of the service, should go to the IP of the Pod. Once that is in place, whenever a POD tries to reach the IP of the service, it is forwarded to the PODs.
Whenever a service is created or deleted, kube-proxy creates or deletes the forwarding rules. They also contain specific source ports and destination ports. To do these forwardings, `userspace`, `iptables` or `ipvs`(ip virtual server) is used. Range to be used for service IPs is set in the Kube-apiserver option called service-cluster-ip-range which is by default (10.0.0.0/24). This range is different from what is set as PODs Ip range and it should always be different. You can check the logs for kube-proxy pod or if it is a service running with systemd it will have logs in `/var/log/kube-proxy.log`

## Endpoints

Endpoints within a service are based off of Endpoints object that is created by using selectors to select all the pods that match a given label.

## DNS

- Generally runs as a Pod in the cluster but doesn't have to. Also it is exposed as a Service VIP, but doesn't have to be.
- Containers are configured by kubelet to use Kube-dns.
- Default implementation is CoreDNS

## Services:DNS

```
my-service.default.svc.cluster.local
```
which basically follows the format:
```
svc-name.namespace.svc.domain
```


# Kube-proxy

- It is default implementation but not the only implementation.
- It runs on every node and uses node as a proxy for trafic from the pods on that node

- It uses:
    - iptables, IPVS, winkernel or userspace options
    - On Linux: iptables and IPVS(IP virtual server) are best choice
- It is transparent to consumers


# Servie LoadBalancers

- Services are also how you configure L4 load-balancers
- Different LBs work in differen ways
- K8s has integration with LBs provided by most cloud providers

# Ingress

- Works at L7
- Describe an HTTP Proxy and routing rules
    - very simple API - match hostnames and url paths
    - too simple in some cases
- Targets a Service for each rule
- K8s defines API but the implementations are 3rd party
- Integrations with most clouds and popular software LBs
- Provides ability to create SSL encrypted routes exposing applications
- You can use Nginx/HAProxy/Traefik as Ingress Controllers(reverse proxies) and Configure Ingress resources in the cluster using K8s yamls
- There is no ingress controller by default and hence fresh K8s won't be able to provide Ingress capabilities until you deploy something

# NetworkPolicy

- Describes the allowed call-graph for communications
    - e.g. frontends can talk to backends, backedns to DB but frontends can't/shouldn't ever talk to DB
- Like ingress, implementations are 3rd party
    - Often highly coupled with low-level network drivers
- Very app-focued for app owners and not for cluster admins



# OpenShift 4.6 Networking

Kubernetes ensures that Pods are able to network with each other, and allocates each Pod an IP address from an internal network. This ensures all containers within the Pod behave as if they were on the same host.

Giving each Pod its own IP address means that Pods can be treated like physical hosts or virtual machines in terms of port allocation, networking, naming, service discovery, load balancing, application configuration, and migration.

OpenShift Container Platform has a built-in DNS so that the services can be reached by the service DNS as well as the service IP/port.


OpenShift 4.6 has Cluster Network operator that helps you define the Cluster Network.

Sample Config:
```yaml
apiVersion: operator.openshift.io/v1
kind: Network
metadata:
  name: cluster
spec:
  clusterNetwork: 
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  serviceNetwork: 
  - 172.30.0.0/16
  defaultNetwork:  # Configures the default CNI network provider for the cluster network.
    ...
  kubeProxyConfig: # If you are using the OVN-Kubernetes default CNI network provider, the kube-proxy configuration has no effect.
    iptablesSyncPeriod: 30s 
    proxyArguments:
      iptables-min-sync-period: 
      - 0s
```

In the above config, we are missing the `defaultnetwork` section that can be defined wuth OpenShift SDN or OVN-Kubernetes as follows:

```yaml
defaultNetwork:
  type: OpenShiftSDN 
  openshiftSDNConfig: 
    mode: NetworkPolicy 
    mtu: 1450 
    vxlanPort: 4789 
```

And OVN-Kubernetes as:
```yaml
defaultNetwork:
  type: OVNKubernetes 
  ovnKubernetesConfig: 
    mtu: 1400 
    genevePort: 6081 
```

## OpenShift SDN



OpenShift Container Platform uses a software-defined networking (SDN) approach to provide a unified cluster network that enables communication between Pods across the OpenShift Container Platform cluster. This Pod network is established and maintained by the OpenShift SDN, which configures an overlay network using Open vSwitch (OVS).

OpenShift SDN provides three SDN modes for configuring the Pod network:

- The `network policy mode` allows project administrators to configure their own isolation policies using NetworkPolicy objects. Network policy is the default mode in OpenShift Container Platform 4.6.

- The `multitenant mode` provides project-level isolation for Pods and Services. Pods from different projects cannot send packets to or receive packets from Pods and Services of a different project. You can disable isolation for a project, allowing it to send network traffic to all Pods and Services in the entire cluster and receive network traffic from those Pods and Services.

- The `subnet mode` provides a flat Pod network where every Pod can communicate with every other Pod and Service. The network policy mode provides the same functionality as the subnet mode.


## OVN Kubernetes in OpenShift

OpenShift 4.6 uses OVN-Kubernetes CNI plug-in as Default network provider for the cluster. OpenShift also offers OpenShift SDN for CNI if you don't want to use OVN-K8s.

### It has following Features:

- Uses Open Virtual Network(OVN) to manage the network traffic flow. It is community developed and vendor agnostic network virtualization solution.
- Implements support for K8s network policies, including Ingress and Egress Rules. OpenShift SDN does not support egress rules and some `ipBlock` rules.
- Uses Geneve (Generic Network Virtualization Encapsulation) protocol rther than VXLAN to create an overlay between nodes.

Kubernetes networking requirements:

OVN is a network virtualization solution. It can create logical switches and routers. One can then interconnect these switches and routers to create any network topologies. Kubernetes (k8s) in its default mode has the following networking requirements.

    All containers should be able to communicate with each other over IP address.

OVN is capable of having overlapping IP addresses (as part of different network topologies). But with the k8s requirement that all containers should be able to talk over IP address, it makes sense to have a simple network topology with one logical switch created per k8s node and then inter-connect all those logical switches with a single logical router. Any containers created in these nodes get connected to the logical switch associated with that node as a logical port.

    All nodes should be able to communicate with the containers running inside that node over IP address.

k8s has this requirement for health-checking. To achieve this goal, we will create an additional logical port for the node (represented by a OVS internal interface).

    All containers should be able to communicate with the k8s daemons running on the master.

This can be achieved by running OVN gateways on the nodes. With at least one OVN gateway, the pods can reach the k8s central daemons with NAT.


## Multiple Networks in OCP

In Kubernetes, container networking is delegated to networking plug-ins that implement the Container Network Interface (CNI).

OpenShift Container Platform uses the Multus CNI plug-in to allow chaining of CNI plug-ins. During cluster installation, you configure your default Pod network. The default network handles all ordinary network traffic for the cluster. You can define an additional network based on the available CNI plug-ins and attach one or more of these networks to your Pods. You can define more than one additional network for your cluster, depending on your needs. This gives you flexibility when you configure Pods that deliver network functionality, such as switching or routing.


OpenShift Container Platform provides the following CNI plug-ins for creating additional networks in your cluster:

- bridge: Creating a bridge-based additional network allows Pods on the same host to communicate with each other and the host.

- host-device: Configuring a host-device additional network allows Pods access to a physical Ethernet network device on the host system.

- ipvlan: Configuring an ipvlan-based additional network allows Pods on a host to communicate with other hosts and Pods on those hosts, similar to a macvlan-based additional network. Unlike a macvlan-based additional network, each Pod shares the same MAC address as the parent physical network interface.

- macvlan: Creating a macvlan-based additional network allows Pods on a host to communicate with other hosts and Pods on those hosts by using a physical network interface. Each Pod that is attached to a macvlan-based additional network is provided a unique MAC address.

    A macvlan additional network can be configured in two ways:

        Configuring a macvlan-based additional network with basic customizations

        Configuring a macvlan-based additional network

- SR-IOV: Configuring an SR-IOV based additional network allows Pods to attach to a virtual function (VF) interface on SR-IOV capable hardware on the host system.



## Required Ports List

Various Components of K8s need following Ports to be opened on the underlying hosts:

ETCDserver: 2379, if there are multiple etcd servers (like if you have 2 masters/etcd servers), etcdClients use 2380 between each other

Kube-ApiServer: 6443

Kubelet: 10250

Kube-Scheduler: 10251

Kube-Controller-Manager: 10252

