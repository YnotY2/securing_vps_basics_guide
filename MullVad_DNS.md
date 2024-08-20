# MullVad DNS Configuration

This README.md file provides an overview of configuring VPS to use custom DNS configuration. I am choosing to use MullVad DNS server's they provide access to for free without subscription. I trust there service fully, and they advertise 
no log policy on there DNS servers. 

## Features DNS Content Blockers
```
Hostname 	            Ads 	Trackers 	Malware 	Adult 	Gambling 	Social media
all.dns.mullvad.net 	✅ 	    ✅ 	        ✅ 	        ✅ 	    ✅ 	        ✅
```

## Table of Contents

1. [Confirgure MullVad DNS](#configure-mullvad-dns)
2. [Configuration Files](#configuration-files)
3. [Configuration Overview](#configuration-overview)
4. [Verify Custom DNS Settings](#verify-custom-dns-settings)
6. [Checking the Logs](#checking-the-logs)
7. [Important Notes](#important-notes)

# Confirgure MullVad DNS
You can simply follow this pretty good guide provided by MullVad VPN: 
- This set-up utilises systemd-resolved

```
https://mullvad.net/en/help/dns-over-https-and-dns-over-tls
```

## Configuration Files

### `/etc/systemd/resolved.conf`

This file is the primary configuration for `systemd-resolved`, which manages DNS resolution on your system. In this file, you specify the DNS servers you want to use. Here’s an example configuration for using MullVad DNS servers:

```
# This file is part of systemd.
#
# systemd is free software; you can redistribute it and/or modify it under the
# terms of the GNU Lesser General Public License as published by the Free
# Software Foundation; either version 2.1 of the License, or (at your option)
# any later version.
#
# Entries in this file show the compile time defaults. Local configuration
# should be created by either modifying this file, or by creating "drop-ins" in
# the resolved.conf.d/ subdirectory. The latter is generally recommended.
# Defaults can be restored by simply deleting this file and all drop-ins.
#
# Use 'systemd-analyze cat-config systemd/resolved.conf' to display the full config.
#
# See resolved.conf(5) for details.

[Resolve]
# Some examples of DNS servers which may be used for DNS= and FallbackDNS=:
# Cloudflare: 1.1.1.1#cloudflare-dns.com 1.0.0.1#cloudflare-dns.com 2606:4700:4700::1111#cloudflare-dns.com 2606:4700:4700::1001#cloudflare-dns.com
# Google:     8.8.8.8#dns.google 8.8.4.4#dns.google 2001:4860:4860::8888#dns.google 2001:4860:4860::8844#dns.google

DNS=194.242.2.9#all.dns.mullvad.net
DNSSEC=no
DNSOverTLS=yes
Domains=~.
```

### `/etc/resolv.conf`

This file is used to direct DNS queries from the local system. When configured with `systemd-resolved`, it should be a symbolic link to `/run/systemd/resolve/stub-resolv.conf`. Here’s what you’ll see if you follow the symlink:

```
# This is /run/systemd/resolve/stub-resolv.conf managed by man:systemd-resolved(8).
# Do not edit.
#
# This file might be symlinked as /etc/resolv.conf. If you're looking at
# /etc/resolv.conf and seeing this text, you have followed the symlink.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0 trust-ad
search .
```

## Configuration Overview

- **`/etc/systemd/resolved.conf`**: Configures the DNS servers and settings for `systemd-resolved`. The `DNS` line specifies the DNS servers to use, such as the MullVad servers. Other settings control DNS security and DNS-over-TLS.
  
- **`/etc/resolv.conf`**: This file is managed by `systemd-resolved` and is symlinked to `/run/systemd/resolve/stub-resolv.conf`. It is dynamically updated by `systemd-resolved` to reflect the DNS servers and configuration you’ve set in `/etc/systemd/resolved.conf`.

Local DNS queries are directed through `systemd-resolved`, which forwards them to the configured DNS servers. The `nameserver 127.0.0.53` entry in `/etc/resolv.conf` points to the local `systemd-resolved` stub resolver, which handles DNS requests and applies the configured settings.

## Verify Custom DNS Settings

To verify that your custom DNS settings are working correctly, you can use several commands to check the current DNS configuration and ensure that it is using the MullVad DNS servers as configured. Follow these steps:

1. **Check DNS Configuration with `resolvectl status`**

   Run the following command to view the current DNS configuration and server details:

   ```
   symbol@srv579320:~$ resolvectl status
   Global
   Protocols: +LLMNR +mDNS +DNSOverTLS DNSSEC=no/unsupported
   resolv.conf mode: stub
   Current DNS Server: 194.242.2.9#all.dns.mullvad.net
   DNS Servers 194.242.2.9#all.dns.mullvad.net
   DNS Domain ~.
    
    Link 2 (eth0)
        Current Scopes: DNS LLMNR/IPv4 LLMNR/IPv6
             Protocols: +DefaultRoute +LLMNR -mDNS +DNSOverTLS DNSSEC=no/unsupported
    Current DNS Server: 194.242.2.9
           DNS Servers: 194.242.2.9
  ``

2. **View the Contents of `/etc/resolv.conf`**

   This file should be symlinked to `/run/systemd/resolve/stub-resolv.conf`. It shows how DNS queries are routed:

   ```
   symbol@srv579320:~$ cat /etc/resolv.conf 
   # This is /run/systemd/resolve/stub-resolv.conf managed by man:systemd-resolved(8).
   # Do not edit.
   #
   # This file might be symlinked as /etc/resolv.conf. If you're looking at
   # /etc/resolv.conf and seeing this text, you have followed the symlink.
   #
   # This is a dynamic resolv.conf file for connecting local clients to the
   # internal DNS stub resolver of systemd-resolved. This file lists all
   # configured search domains.
   #
   # Run "resolvectl status" to see details about the uplink DNS servers
   # currently in use.
   #
   # Third party programs should typically not access this file directly, but only
   # through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
   # different way, replace this symlink by a static file or a different symlink.
   #
   # See man:systemd-resolved.service(8) for details about the supported modes of
   # operation for /etc/resolv.conf.

   nameserver 127.0.0.53
   options edns0 trust-ad
   search .
   ```

3. **Verify DNS Resolution with `dig`**

   Use the `dig` command to confirm that DNS queries are being resolved correctly through the configured DNS server:
   - You should see the 'localhost' server, as this is where we foward requests.

   ```
   symbol@srv579320:~$ dig example.com

   ; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> example.com
   ;; global options: +cmd
   ;; Got answer:
   ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1186
   ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

   ;; OPT PSEUDOSECTION:
   ; EDNS: version: 0, flags:; udp: 65494
   ;; QUESTION SECTION:
   ;example.com.			IN	A

   ;; ANSWER SECTION:
   example.com.		2964	IN	A	93.184.215.14

   ;; Query time: 0 msec
   ;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
   ;; WHEN: Tue Aug 20 12:33:17 UTC 2024
   ;; MSG SIZE  rcvd: 56
   ```

5. **Re-check `/etc/systemd/resolved.conf`**

   Ensure that the configuration file has the correct settings for the DNS servers:

   ```
   symbol@srv579320:~$ cat /etc/systemd/resolved.conf 
   # This file is part of systemd.
   #
   # systemd is free software; you can redistribute it and/or modify it under the
   # terms of the GNU Lesser General Public License as published by the Free
   # Software Foundation; either version 2.1 of the License, or (at your option)
   # any later version.
   #
   # Entries in this file show the compile time defaults. Local configuration
   # should be created by either modifying this file, or by creating "drop-ins" in
   # the resolved.conf.d/ subdirectory. The latter is generally recommended.
   # Defaults can be restored by simply deleting this file and all drop-ins.
   #
   # Use 'systemd-analyze cat-config systemd/resolved.conf' to display the full config.
   #
   # See resolved.conf(5) for details.

   [Resolve]
   # Some examples of DNS servers which may be used for DNS= and FallbackDNS=:
   # Cloudflare: 1.1.1.1#cloudflare-dns.com 1.0.0.1#cloudflare-dns.com 2606:4700:4700::1111#cloudflare-dns.com 2606:4700:4700::1001#cloudflare-dns.com
   # Google:     8.8.8.8#dns.google 8.8.4.4#dns.google 2001:4860:4860::8888#dns.google 2001:4860:4860::8844#dns.google

   DNS=194.242.2.9#all.dns.mullvad.net
   DNSSEC=no
   DNSOverTLS=yes
   Domains=~.
   ```

Ensure that all these checks confirm that your custom DNS settings are properly configured and active. Adjust any settings if necessary and verify again.


   
4. **Restart Fail2Ban Service**
    After making changes, restart Fail2Ban to apply them:

    ```shell
    sudo systemctl restart fail2ban
    ```

3. **Check Status**
    Verify the status of Fail2Ban and specific jails:

    ```shell
    fail2ban-client status
    ```

    Check the status of a specific jail:

    ```shell
    fail2ban-client status sshd
    ```

# 

# Checking the Logs `/var/log/fail2ban.log`

Fail2Ban logs critical information about its operations and detected threats in the log file located at `/var/log/fail2ban.log`. This file is essential for monitoring Fail2Ban’s performance and troubleshooting issues.

## Accessing the Log File
Utilise cat or nano to view the file, very usefull for viewing if a test-ban has indeed been rejected. Or viewing live bans from server access via ssh. 

## Important Notes

- **Do Not Modify Default Files**: `fail2ban.conf` and `jail.conf` should not be modified directly. Instead, use `fail2ban.local` and `jail.local` for customizations.
- **Review Documentation**: Refer to the Fail2Ban man pages for detailed configuration options and advanced features.
