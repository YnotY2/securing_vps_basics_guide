# User Management for VPS 

This guide provides instructions on how to add a new user and grant them sudo privileges on your VPS.

## Add a New User

To create a new user, use the adduser command. Replace newusername with the desired username:

```bash
sudo adduser newusername
```

You will be prompted to set a password and optionally provide additional information (such as Full Name). Fill these out as needed.

## Add the User to the Sudo Group

### On Ubuntu or Debian-based Systems

Users need to be added to the sudo group to gain sudo privileges:

```bash
sudo usermod -aG sudo newusername
```
## Verification

To verify that the new user has sudo access, switch to the new user account:

```bash
su - newusername
```

Test sudo access by running a command with sudo:

```bash
sudo su
```

It should return root, indicating that the user has sudo privileges.
