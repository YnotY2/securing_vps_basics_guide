# Securing SSH access 
This guide will assume you have already disabled root login for the VPS. It will focus on setting up a key-pair and aslo a extra protective seedphrase when loggin in to VPS.
- This guide assumes you have already created a fresh user on the VPS, and added it to the su group.
- *Use the created user on you're VPS not the root user*

# Creating SSH key-pair

We will create the private and public key-pair within our custom directory path.
```
ssh-keygen -t ed25519 -C "My VPS Key" -f /path/to/your/custom_directory/custom_filename
```

e.g;
```
🔗 ssh-keygen -t ed25519 -C "My VPS Key" -f /home/mykeys/ssh/vps/main_vps_key
```


# Moving public key-pair to VPS

We will store our public-key generated from the pair within a standard directory on the VPS. 
- Login to you're VPS with the created user on the VPS 

1. **Copy** the *public_key*
- The Public-Key will end within `.pub` 
```
cat /path/to/your/custom_directory/custom_filename.pub
```

2. **Create** the *~/.ssh* directory
```
mkdir -p ~/.ssh
```

3. **Grant correct permissions** to the *~/.ssh* directory
```
chmod 700 ~/.ssh
```

4. **Create and Add** *public-ssh-key* (.pub) to file 'authorized_keys' for storing the ssh public key (from the pair created earlier) within VPS
```
sudo nano ~/.ssh/authorized_keys
```

5. **Grant correct permissions** to the *~/.ssh/authorized_keys* directory
```
chmod 600 ~/.ssh/authorized_keys
```

6. **Restart** *ssh and sshd* systemctl service, to **apply changes**
```
sudo systemctl restart sshd
```
```
sudo systemctl restart sshd
```

7. **Check** the *status*  of both services, to *confirm correct restart*
```
sudo systemctl status sshd
```
```
sudo systemctl status sshd
```


# Modify the SSHD Config File
The sshd config file authenticates the inbound client side ssh connections. Because we are running ssh on the server side, this congiguration file is used for authentication of a incomming client.
- **Location:** Typically found in */etc/ssh/sshd_config* for system-wide settings.

We will be 



# Confirm ssh login to VPS

**Confirm** *SSH Key-Based Login* works as expected
- From you're localhost outside of VPS confirm login.
```
ssh -i /path/to/your/private_key yourusername@your_vps_ip
```

e.g;

```
🔗 ssh -i /home/mykeys/ssh/vps/main_vps_key yourusername@your_vps_ip

```

7. 
