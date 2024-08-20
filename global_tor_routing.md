# TOR Configuration
This is a quick guide to setting up TOR on a VPS, no complex routing rules with nftables. Just simply setting up basic tor usage, routing utilising torify. Monitoring TOR interface, traffic and settings using 'nyx' aswell.
Traffic is from locahost is routed trough a socks5 proxy. 

TOR routes traffic from localhost to maintain privacy, control, and flexibility. This setup ensures that traffic is anonymized before it reaches the internet, prevents exposure of the public IP address, and allows for selective routing of traffic through TOR. It is a well-established practice in TOR’s design and usage to balance security and practicality.

- *If you want to route all outbound traffic over tor checkout 'global_tor_routing.md'*



## Table of Contents

0. [Assumptions Set-Up](#Aassumptions-set-up)
2. [Confirm TOR Default Settings](#confirm-tor-default-settings)
    - [1. Check TOR Service Status](#check-tor-service-status)
    - [2. Test TOR Connection](#test-tor-connection)
3. [Modifying TOR confiration file](#modifying-tor-configuration-file)
4. [nftables](#nftables)
    - [Configuration File](#configuration-file-global_tor_routing.nft)
    - [Configuring System Interface](#configuring-system-interface)
    - [Unique Values](#unique-values)
    
# Assumptions Set-Up
This guide assumes you have followed 'configure_tor.md', and confirmed TOR connection works.



# Modifying TOR confiration file
We will be added a few rules that are needed for us to be able to route all traffic over 
TOR. These are very usefull especially when configuring with nftables firewall.

# `/etc/tor/torrc file`
```
SocksPort 9050
29 VirtualAddrNetworkIPv4 10.192.0.0/10
TransPort 9040 IsolateClientAddr IsolateClientProtocol IsolateDestAddr IsolateDestPort
AutomapHostsOnResolve 1
DNSPort 5353
ControlPort 9051
HashedControlPassword 16:16:01212Ff122122121DF3141E15EXAMPLHASHJ

```

## *Enabled Settings/Rules*

- **SocksPort 9050**
  - *Port TOR service is listening on for outbound connections to be routed.*

- **VirtualAddrNetworkIPv4 10.192.0.0/10**
  - *Specifies a range of virtual IPv4 addresses that Tor will use to map client addresses when routing traffic through the network. This is particularly useful for handling connections that don't use a direct IP address (e.g., resolving DNS).*

- **TransPort 9040 IsolateClientAddr IsolateClientProtocol IsolateDestAddr IsolateDestPort**
  - *Port TOR service is listening on for outbound transparent proxy connections, isolating streams based on client address, protocol, destination address, and destination port. This ensures greater privacy by making it more difficult to correlate traffic.*
  
- **AutomapHostsOnResolve 1**
  - *Automatically maps and resolves hostnames to an IP address within the virtual address network defined by `VirtualAddrNetworkIPv4`. This is useful for dealing with DNS requests in a privacy-preserving manner.*

- **DNSPort 5353**
  - *Port on which Tor will listen for DNS requests. When a DNS request is made, it will be routed through the Tor network, ensuring that DNS queries are anonymized.*

- **ControlPort 9051**
  - *Port for the Tor control protocol, which allows a user or application to communicate with the Tor process for status checks, configuration changes, or other administrative functions.*

- **HashedControlPassword 16:01212Ff122122121DF3141E15EXAMPLHASH**
  - *This hashed password is used to authenticate clients attempting to connect to the Tor control port. The password is hashed for security, ensuring that the actual password isn't exposed in the configuration file.*

  You can just copy the above listed torrc file, the only thing needed to be modified is you're own Controll Port Password. (which you should already have set-up)

 **Restart TOR**

   Apply the changes by restarting the TOR service:

   ```
   sudo systemctl restart tor
   ```

   When prompted, enter the password you configured. If everything is set up correctly, `nyx` will connect to the TOR controller and display the monitoring interface. Here you can verify the extra rules/lines you have enabled within /etc/tor/torrc are     displayed and working. 



# Nftables
Nftables is a very powerfull modern firewall, via cli. In this specific use-case we are going to configure the managing of traffic flow in a system that uses the Tor network for anonymity, forcing all outbound traffic trough TOR network aka nodes. We enforce strict controls on which traffic is allowed in and out of the system. The firewall setup involves two tables: nat for network address translation and filter for packet filtering.


## Coniguration File 'global_tor_routing.nft'
```
# Verify your network interface with ip addr
define interface = eth0
# Verify tor uid with id -u tor, in debain find it using htop, for me it's: debian-tor
define uid = 105


table ip nat {
        set unrouteables {
                type ipv4_addr
                flags interval
                elements = { 127.0.0.0/8, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 0.0.0.0/8, 100.64.0.0/10, 169.254.0.0/16, 192.0.0.0/24, 192.0.2.0/24, 192.88.99.0/24, 198.18.0.0/15, 198.51.100.0/24, 203.0.113.0/24, 224.0.0.0/4, 240.0.0.0/4 }
        }

        chain POSTROUTING {
                type nat hook postrouting priority 100; policy accept;
        }

        chain OUTPUT {
                type nat hook output priority -100; policy accept;
                meta l4proto tcp ip daddr 10.192.0.0/10 redirect to :9040
                meta l4proto udp ip daddr 127.0.0.1 udp dport 53 redirect to :5353
                skuid $uid return
                oifname "lo" return
                ip daddr unrouteables return
                meta l4proto tcp redirect to :9040
        }
}
table ip filter {
        set private {
                type ipv4_addr
                flags interval
                elements = { 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 127.0.0.0/8 }
        }
        chain INPUT {
                type filter hook input priority 0; policy drop;
                # Allow Local SSH connections
                iifname $interface meta l4proto tcp tcp dport 22 ct state new accept
                ct state established accept
                iifname "lo" accept
                ip saddr private accept
        }

        chain FORWARD {
                type filter hook forward priority 0; policy drop;
        }

        chain OUTPUT {
                type filter hook output priority 0; policy drop;
                ct state established accept
                oifname $interface meta l4proto tcp skuid $uid ct state new accept
                oifname "lo" accept
                ip daddr private accept
        }
}
```


## Unique Values
Within the firewall configurations there are a few unique requirements necessary for the firewall to work outside of
just having TOR running with all the listening PORT working. A internal I.P on a internal interface is needed. This kinda
acts like a bridge to foward traffic over internally.

1. Internal I.P on fresh interface
# Virtual Network Interface Setup

This guide provides instructions for creating a virtual network interface on both Debian/Ubuntu-based and Red Hat/CentOS-based systems. 

## 1. Create a Virtual Network Interface

On many modern Linux distributions, interfaces are typically managed by the system’s network manager or by configuration files. However, if you need to manually create a new virtual interface, you can use the following approach. This is usually more about configuring a virtual interface rather than creating one from scratch, as the creation of interfaces might depend on the VPS provider's setup.

### Using Network Configuration Tools

#### For Debian/Ubuntu-based Systems:

**Using `ip` command:**

If you’re creating a virtual interface (e.g., `eth1`), you can use the `ip` command to configure it.

```bash
sudo ip link add link eth0 name eth1 type macvlan mode bridge
```

Here’s what this does:
- `link eth0`: Specifies that `eth0` is the parent interface.
- `name eth1`: Creates a new virtual interface named `eth1`.
- `type macvlan mode bridge`: Configures it as a MACVLAN in bridge mode.

**Bring Up the Interface:**

```bash
sudo ip link set eth1 up
```

## 2. Assign an IP Address to the New Interface

Once you have the interface (e.g., `eth1`) set up, you can assign an IP address to it.

```bash
sudo ip addr add 192.168.1.10/24 dev eth1
```

**Bring Up the Interface:**

```bash
sudo ip link set dev eth1 up
```

## 3. Confirm the Interface is Up and Accessible

To verify that the interface is functioning correctly, you can ping the IP address assigned to it.

```bash
ping 192.168.1.10
```



In this setup, the internal IP on an internal interface acts as a bridge or gateway to manage traffic between different network segments. This internal interface can handle traffic between the local system and the Tor network, effectively isolating internal and external traffic flows.








