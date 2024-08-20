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
4. 
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

