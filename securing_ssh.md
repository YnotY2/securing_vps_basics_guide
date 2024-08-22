# Securing SSH access 
This guide will assume you have already disabled root login for the VPS. It will focus on setting up a key-pair and aslo a extra protective seedphrase when loggin in to VPS.
- This guide assumes you have already created a fresh user on the VPS, and added it to the su group.
- *adding_user_vps.md*
- *Use the created user on you're VPS not the root user*

## Features SSH
-   Disable root login ✅
-   Disable password based auth ✅
-   Enabled ssh-key pair auth only ✅
-   Contact client to check if it is still alive every 120s ✅
-   Disable any outbound ssh conntions from server, it does not act as a client ✅
-   Display of the message of the day (MOTD) upon login ✅
    - ```
      Linux srv579320 6.1.0-23-cloud-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.99-1 (2024-07-15) x86_64
      
      The programs included with the Debian GNU/Linux system are free software;
      the exact distribution terms for each program are described in the
      individual files in /usr/share/doc/*/copyright.
      
      Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
      permitted by applicable law.
      Last login: Thu Aug 22 16:22:55 2024 from 193.32.249.138

      ```

   - [Securing SSH Access](#securing-ssh-access)
   - [Features SSH](#features-ssh)
   - [Creating SSH Key-Pair](#creating-ssh-key-pair)
   - [Moving Public Key-Pair to VPS](#moving-public-key-pair-to-vps)
   - [Modify the SSHD Config File](#modify-the-sshd-config-file)
   - [Adding SSH Private-Key to SSH-Keyring](#adding-ssh-private-key-to-ssh-keyring)
   - [Confirm SSH Login to VPS](#confirm-ssh-login-to-vps)


# Creating SSH key-pair

We will create the private and public key-pair within our custom directory path.
```
ssh-keygen -t ed25519 -C "My VPS Key" -f /path/to/your/custom_directory/custom_filename
```

e.g;
```
🔗 ssh-keygen -t ed25519 -C "My VPS Key" -f /home/mykeys/ssh/vps/main_vps_key
```


## Moving public key-pair to VPS

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


## Modify the SSHD Config File
The sshd config file authenticates the inbound client side ssh connections. Because we are running ssh on the server side, this congiguration file is used for authentication of a incomming client.
- **Location:** Typically found in */etc/ssh/sshd_config* for system-wide settings.
- *We will be modifying the file to optimise security.*

1. **Open** sshd_config file located at *`/etc/ssh/sshd_config`*
```
sudo nano /etc/ssh/sshd_config
```

2. **Add the following lines**
```
UsePAM yes
PrintMotd yes
KbdInteractiveAuthentication no

# Authorized Keys
AuthorizedKeysFile .ssh/authorized_keys

ClientAliveInterval 120
PermitRootLogin no
PasswordAuthentication no
```


### **Lines/Rules Explained:**

1. **`AuthorizedKeysFile .ssh/authorized_keys`**
   - **Explanation:** This line specifies the file that contains the public keys for SSH authentication. It tells the SSH server where to look for the authorized keys for each user. By default, this is set to `.ssh/authorized_keys`, which is a hidden directory and file in the user’s home directory.

2. **`ClientAliveInterval 120`**
   - **Explanation:** This setting controls how often (in seconds) the SSH server will send a message to the client to check if it is still alive. If the client does not respond within the specified time, the server may close the connection. In this case, it’s set to 120 seconds, meaning the server will check the client’s status every 2 minutes.

3. **`PermitRootLogin no`**
   - **Explanation:** This option disables root login over SSH. It is a security measure to prevent direct access to the root account via SSH, which can help protect against unauthorized access and brute-force attacks. Instead, users should log in with a regular account and use `sudo` or `su` to execute commands with elevated privileges.

4. **`PasswordAuthentication no`**
   - **Explanation:** This setting disables password-based authentication for SSH. By setting this to `no`, SSH will only allow key-based authentication (using SSH keys). This is generally considered more secure than password authentication, as it mitigates the risk of brute-force attacks on passwords.

5. **`PrintMotd yes`**
   - **Explanation:** Display of the message of the day (MOTD) upon login. This is useful for reducing unnecessary output and focusing on essential information.

6. **`KbdInteractiveAuthentication no`**
   - **Explanation:** Disables keyboard-interactive authentication, which is often used for challenge-response passwords. This ensures that only key-based authentication is used.

7. **`UsePAM yes`**
   - **Explanation:** Enables PAM (Pluggable Authentication Modules) for authentication, account processing, and session management. This is necessary if your system relies on PAM for managing authentication or session features.


## Adding ssh private-key to ssh-keyring
To use the SSH key pair you’ve created, you need to add the private key to your SSH keyring. Think of a key pair like a physical lock and key. In this analogy:

 - **The public key** is like the *lock.* It is placed on the server (VPS) where you want to gain access.
 - **The private key** is like the *key* that opens the lock. You keep this private key on your local machine.

When you connect to the server, the SSH client uses the private key to “unlock” the public key on the server, completing a secure handshake.
-You need to run the following cli cmd to add private-key to ssh-keyring
```
sudo -E ssh-add /path/to/key_created/private_key
```

e.g;

```
🔗 sudo -E ssh-add /home/mykeys/ssh/vps/main_vps_key
```


## Confirm ssh login to VPS

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
