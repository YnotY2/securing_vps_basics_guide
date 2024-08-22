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
