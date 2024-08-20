# TOR Configuration
This is a quick guide to setting up TOR on a VPS, no complex routing rules with nftables. Just simply setting up basic tor usage, routing utilising torify. Monitoring TOR interface, traffic and settings using 'nyx' aswell.
Traffic is from locahost is routed trough a socks5 proxy. 

TOR routes traffic from localhost to maintain privacy, control, and flexibility. This setup ensures that traffic is anonymized before it reaches the internet, prevents exposure of the public IP address, and allows for selective routing of traffic through TOR. It is a well-established practice in TOR’s design and usage to balance security and practicality.

## Table of Contents

1. [Installation of Requirements](#installation-of-requirements)
    - [Update System Packages](#update-system-packages)
    - [1. Install TOR](#install-tor)
    - [2. Install torsocks](#install-torsocks)
    - [3. Install nyx](#install-nyx)
2. [Confirm TOR Default Settings](#confirm-tor-default-settings)
    - [1. Check TOR Service Status](#check-tor-service-status)
    - [2. Test TOR Connection](#test-tor-connection)
3. [Enable Nyx Monitoring Interface](#enable-nyx-monitoring-interface)
    - [1. Create a TOR Controller Password Hash](#create-a-tor-controller-password-hash)
    - [2. Configure TOR Controller Settings](#configure-tor-controller-settings)
    - [3. Restart TOR](#restart-tor)
    - [4. Verify Nyx Functionality](#verify-nyx-functionality)


# Installation of Requirements

This document outlines the steps to install the necessary software packages via the command line interface (CLI).

0. **Update System Packages**

First, update the package index on your system to ensure you have the latest information:

```
sudo apt update
```

1. **Install TOR**

TOR (The Onion Router) is free software for enabling anonymous communication. It directs Internet traffic through a worldwide volunteer network to protect user privacy and anonymity.

```
sudo apt install tor
```

2. **Install torsocks**

torsocks is a wrapper that allows network applications to use TOR for routing their traffic. It helps ensure that the applications' network traffic is anonymized through the TOR network.

```
sudo apt install tor torsocks
```

3. **Install nyx**

nyx (formerly known as arm) is a command-line tool used for monitoring the TOR network and your local TOR relay. It provides a real-time and detailed view of your TOR node's performance and status.

```
sudo apt install nyx
```

## Confirm TOR Default Settings

To ensure that TOR is running with its default settings, follow these steps:

1. **Check TOR Service Status**

Verify that the TOR service is active and running by executing the following command:

```
sudo systemctl status tor
```

This command will display the current status of the TOR service, including whether it is active, running, or encountering any issues.

2. **Test TOR Connection**

To confirm that TOR is functioning correctly, you can test its network routing by making a request through TOR's SOCKS proxy. Use the `curl` command to perform a DNS leak test via TOR:

```
curl --socks5 127.0.0.1:9050 https://dnsleaktest.com
```

This command sends a request to `dnsleaktest.com` using the TOR SOCKS proxy running on `127.0.0.1:9050`. If TOR is properly configured, you should see a web page output from the DNS leak test site.


## Enable Nyx Monitoring Interface

To set up the `nyx` monitoring interface for TOR, follow these steps:

1. **Create a TOR Controller Password Hash**

   First, generate a password hash for the TOR controller. This password will be used to authenticate with the `nyx` monitoring interface. Run the following command to create a hashed password:

   ```
   tor --hash-password yourpassword
   ```

   Replace `yourpassword` with a secure password of your choice. Make a note of the generated hash value as you will need it in the next steps.

2. **Configure TOR Controller Settings**

   Edit the TOR configuration file to enable the controller port and set the password hash. Open the TOR configuration file in a text editor:

   ```
   sudo nano /etc/tor/torrc
   ```

   Add the following lines to the configuration file:

   ```
   ControlPort 9051
   HashedControlPassword yourhashedpassword
   ```

   Replace `yourhashedpassword` with the hash value you obtained in the previous step. Save the changes and exit the text editor (in Nano, press `Ctrl+X`, then `Y`, and `Enter`).

3. **Restart TOR**

   Apply the changes by restarting the TOR service:

   ```
   sudo systemctl restart tor
   ```

4. **Verify Nyx Functionality**

   Confirm that `nyx` is working correctly by running the application and logging in with the password you set. Start `nyx` with the following command:

   ```
   nyx
   ```

   When prompted, enter the password you configured. If everything is set up correctly, `nyx` will connect to the TOR controller and display the monitoring interface.

By following these steps, you will enable and configure the `nyx` monitoring interface to work with your TOR setup.
