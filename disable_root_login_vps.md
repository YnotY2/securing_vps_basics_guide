# Disabling Root Login via SSH

This guide provides step-by-step instructions on how to disable SSH root login on your VPS to enhance security.

## Prerequisites

- Access to your VPS with a user account that has sudo privileges.
- An SSH client to connect to your VPS.

## Steps to Disable Root Login

1. **Connect to Your VPS**

   Use SSH to log in to your VPS with a user account that has sudo privileges:

   @```bash
   ssh root@your_vps_ip
   @```

   Replace `your_username` with your current non-root username and `your_vps_ip` with your VPS’s IP address.

2. **Edit the SSH Configuration File**

   Open the SSH configuration file using a text editor. Here we use `nano`:

   @```bash
   sudo nano /etc/ssh/sshd_config
   @```

   Alternatively, you can use `vi` or another text editor if preferred.

3. **Modify the Configuration**

   Find the line that begins with `PermitRootLogin`. It may be commented out with a `#` symbol or set to `yes`.

   Change or add the following line to disable root login:

   @```bash
   PermitRootLogin no
   @```

   If the line is commented out (i.e., starts with `#`), remove the `#` to uncomment it.

4. **Save and Exit**

   - In `nano`, press `CTRL + X` to exit, then press `Y` to confirm changes, and `Enter` to save.
   - In `vi`, press `ESC`, then type `:wq` and press `Enter`.

5. **Restart the SSH Service**

   For the changes to take effect, restart the SSH service:

   @```bash
   sudo systemctl restart sshd
   @```

   Or, if using `service`:

   @```bash
   sudo service ssh restart
   @```

6. **Verify the Change**

   Open a new terminal session and attempt to log in as root to ensure that root login is disabled:

   ```bash
   ssh root@your_vps_ip
   ```
   You should receive a message indicating that root login is prohibited.


7. **Confirm user login**

  Confirm you can log in with created user creds:
  ```bash
 ssh new_user@your_vps_ip
 ```


## Additional Notes

- Ensure you have a non-root user with `sudo` privileges to manage your VPS.
- Consider setting up SSH key-based authentication for added security.

By following these steps, you will have successfully disabled SSH root login on your VPS, enhancing the security of your system.

---

For more information, refer to the [OpenSSH documentation](https://www.openssh.com/manual.html).
