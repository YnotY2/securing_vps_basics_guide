# Securing SSH access 
This guide will assume you have already disabled root login for the VPS. It will focus on setting up a key-pair and aslo a extra protective seedphrase when loggin in to VPS.


# Creating SSH key-pair

We will create the private and public key-pair within our custom directory path.
```
ssh-keygen -t ed25519 -C "My VPS Key" -f /path/to/your/custom_directory/custom_filename
```

e.g;
```
🔗 ssh-keygen -t ed25519 -C "My VPS Key" -f /home/mykeys/ssh/vps/main_vps_key
```


