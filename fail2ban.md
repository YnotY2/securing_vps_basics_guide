# Fail2Ban Configuration

This README.md file provides an overview of Fail2Ban configuration and its current status. Fail2Ban is a service that scans log files for security threats and bans IP addresses based on predefined rules. This document summarizes the status and configuration of Fail2Ban, including specific jails, configuration files, and actions.

## Current Status

Fail2Ban Client Status for sshd

@```shell
fail2ban-client status sshd
@```

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

- `fail2ban.conf`: Main configuration file, typically not modified directly.
- `fail2ban.local`: For user-defined customizations.
- `jail.conf`: Default configuration for jails.
- `jail.local`: User-defined configurations for jails, where modifications should be made.

## Configuration Overview

Fail2Ban is designed to enhance security by banning IP addresses that exhibit malicious behavior. Configuration options include:

### [DEFAULT] Settings *`/ect/fail2ban/jail.locak`* :

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
    - backend: systemd

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

# Copy `` and `` file content into new files

1. Navigate to the *`/ect/fail2ban/`* directory

2. copy contents of `./jail.conf` into new file `jail.local` :
```
sudo cp ./jail.conf ./jail.local
```
3. copy centens of `./fail2ban.conf` into `fail2ban.local` :
```
sudo cp ./fail2ban.conf ./fail2ban.local
```

1. **Edit Configuration Files**
    - Make necessary changes in `jail.local`.
    - Customize settings for jails and actions based on your requirements.

2. **Restart Fail2Ban Service**
    After making changes, restart Fail2Ban to apply them:

    @```shell
    sudo systemctl restart fail2ban
    @```

3. **Check Status**
    Verify the status of Fail2Ban and specific jails:

    @```shell
    fail2ban-client status
    @```

    Check the status of a specific jail:

    @```shell
    fail2ban-client status sshd
    @```

# 

## Important Notes

- **Do Not Modify Default Files**: `fail2ban.conf` and `jail.conf` should not be modified directly. Instead, use `fail2ban.local` and `jail.local` for customizations.
- **Review Documentation**: Refer to the Fail2Ban man pages for detailed configuration options and advanced features.

For further assistance, consult the Fail2Ban official documentation or seek help from the community forums.
