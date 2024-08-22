# basic_securing_debian_vps
This is as guide to secure a VPS running Debain Latest OS. Making it more secure.


# Login to the VPS
You can either login with pre-configured ssh-key pair or simply with root password for user of  the VPS
```
ssh <root>@<VPS_IP.ADDR.ES.S>
```

## Create a new user
Create a user with admin priviledges for managing the system

- `adding_user_vps.md`

## Secure SSH connection
Securing SSH connections incomming to VPS

- 'securing_ssh.md'

## Secure DNS server
MullVad DNS is a strusted server, I believe in the security and annonimity of the provider

-`MullVad_DNS.md`

## Configurig htop
htop is a very valueble program for monitoring running processes within the system of you're VPS

-`htop_vps.md`

## Securing SSH brute-force attacks
This tool allows for monitoring incomming ssh connections, and will time-out a I.P after certain specified times from attemptign VPS login

-`fail2ban.md`

## Configure TOR 
This tool is very usefull for remaining secure and annonymous on the internet 

-`configure_tor.md`

## Configure Nftables Firewall
This firewall for allow access to server for modifying system, while all traffic is forced through tor exit node. HTTPS and HTTP traffic is allowed over the firewall currently. 

-`global_tor_routing.md`

## Running Python Code
This guide shows how to run python3 code from a VPS

-`running_python_code.md`

