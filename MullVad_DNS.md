# MullVad DNS Configuration

This README.md file provides an overview of configuring VPS to use custom DNS configuration. I am choosing to use MullVad DNS server's they provide access to for free without subscription. I trust there service fully, and they advertise 
no log policy on there DNS servers. 


## Table of Contents

1. [Confirgure MullVad DNS](#configure-mullvad-dns)
2. [Current Status](#current-status)
3. [Configuration Files](#configuration-files)
4. [Configuration Overview](#configuration-overview)
    - [DEFAULT Settings](#default-settings)
    - [Jails Configuration](#jails-configuration)
    - [Actions](#actions)
5. [Steps to Modify Configuration](#steps-to-modify-configuration)
    - [Copy and Create Files](#copy-and-create-files)
    - [Edit Configuration Files](#edit-configuration-files)
    - [Restart Fail2Ban Service](#restart-fail2ban-service)
    - [Check Status](#check-status)
6. [Checking the Logs](#checking-the-logs)
7. [Important Notes](#important-notes)

# Confirgure MullVad DNS
You can simply follow this pretty good guide provided by MullVad VPN: 
- This set-up utilises systemd-resolved

```
https://mullvad.net/en/help/dns-over-https-and-dns-over-tls
```

## Current Status

Fail2Ban Client Status for sshd

```shell
fail2ban-client status sshd
```

Status for the jail: sshd

- **Filter**
    - Currently failed: <example>0
    - Total failed: <amount_example 4>
    - Journal matches: _SYSTEMD_UNIT=sshd.service + _COMM=sshd

- **Actions**
    - Currently banned: <amount_example 1>
    - Total banned: <amount_example 7>
    - Banned IP list: <example_ip 11.442.13.12>

## Configuration Files

The Fail2Ban configuration is distributed across multiple files. The main configuration files include:

- `/etc/systemd/resolved.conf`: Main DNS configuration file, this is where we paste the MullVad DNS servers to use.

```
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it under the
#  terms of the GNU Lesser General Public License as published by the Free
#  Software Foundation; either version 2.1 of the License, or (at your option)
#  any later version.
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
- `/etc/resolv.conf`: This is used to direct DNS queries from localhost. So now that we have configured '/etc/systemd/resolved.conf' it should foward dns routes to custom DNS MullVad server.

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

Fail2Ban is designed to enhance security by banning IP addresses that exhibit malicious behavior. Configuration options include:

### [DEFAULT] Settings *`/ect/fail2ban/jail.local`* :

- **bantime**: Duration for which an IP is banned (default: 10m).
- **findtime**: Time window for tracking failed attempts (default: 10m).
- **maxretry**: Number of allowed failed attempts before banning (default: 5).
- **backend**: Method for monitoring log file changes (default: auto).
- **usedns**: Determines how hostnames are resolved (default: warn).

### Jails Configuration

Jails define the specific services and patterns that Fail2Ban monitors. Some examples include:

- **[sshd]**: Monitors SSH login attempts.
    - enabled: true
    - port: ssh
    - logpath: Path to SSH log file.
    - backend: auto

- **[apache-auth]**: Monitors Apache authentication attempts.
    - port: http, https
    - logpath: Path to Apache error log.

- **[nginx-http-auth]**: Monitors Nginx HTTP authentication attempts.
    - port: http, https
    - logpath: Path to Nginx error log.

### Actions

Actions define what happens when an IP is banned. Default actions include:

- **banaction**: Defines the banning method, e.g., iptables-multiport.
- **action**: Combines banning with optional notifications (e.g., email notifications).

## Steps to Modify Configuration

1. **Copy `` and `` file content into new files**

1. Navigate to the *`/ect/fail2ban/`* directory

2. copy contents of `./jail.conf` into new file `jail.local` :
```
sudo cp ./jail.conf ./jail.local
```
3. copy centens of `./fail2ban.conf` into `fail2ban.local` :
```
sudo cp ./fail2ban.conf ./fail2ban.local
```

2. **Edit Configuration Files**
    - Make necessary changes in `jail.local`.
    - Customize settings for jails and actions based on your requirements.


To customize Fail2Ban’s behavior, you may need to modify the `jail.local` configuration file. This file overrides the default settings specified in `jail.conf` and is used to tailor Fail2Ban's functionality to your specific needs.

### Configuration Entries

1. **bantime**: Sets the duration for which an IP address is banned. Default is `10m` (10 minutes).

2. **findtime**: Defines the time window during which failed attempts are counted. Default is `10m` (10 minutes).

3. **maxretry**: Specifies the number of allowed failed attempts before an IP is banned. Default is `5`.

4. **maxmatches**: Uses the same value as `maxretry` to determine the threshold for banning.

5. **backend**: Determines the method for monitoring log file changes. Default is `auto`.

6. **logencoding**: Sets the encoding for log files. Default is `auto`.

7. **protocol**: Defines the protocol used for monitoring. Default is `tcp`.

8. **enabled**: Activates or deactivates the jail. Set to `true` to enable.

9. **ignoreip**: Specifies IP addresses that should be ignored by Fail2Ban. Default is `127.0.0.1` (localhost).

10. **port**: Defines the port to monitor. For SSH, use `ssh`.

11. **mode**: Specifies the mode for the jail. Default is `normal`.

12. **logpath**: Defines the path to the SSH log file. Use the variable `%(sshd_log)s` to reference the log file path.

### Example `jail.local` Configuration

Below is an example of how these settings should be added to your `jail.local` file:

```ini
[sshd]
bantime  = 10m
findtime  = 10m
maxretry = 5
maxmatches = %(maxretry)s
backend = auto
logencoding = auto
protocol = tcp
enabled = true
ignoreip = 127.0.0.1
port    = ssh
mode   = normal
logpath = %(sshd_log)s
```
   
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
