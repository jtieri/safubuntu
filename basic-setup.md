# Privileged User and Basic Security Configuration

## User Setup
---

Create a new user `anon` and add that user to the `sudo` group for admin privlieges.
```bash
adduser anon
usermod -aG sudo anon
```

## Basic Firewall Setup
---

It's good practice to enable a basic firewall that blocks all incoming connections besides for SSH 

```bash
ufw allow OpenSSH
ufw enable
ufw status
```

## Enable SSH Access Only
---

All connections to the remote server should be made over SSH with a public key. Once a privileged user account is setup configure the SSH keys and then disable password based login.

### Create SSH Key
---

If necessary generate a new key pair on the client machine, `-b 4096` will create a 4096 bit RSA key pair.

```bash
ssh-keygen -b 4096
```

The default location will be at ~/.ssh/ or you can specify an alternate path.

If desired a passphrase for this key can be specified. This is recommended since it adds an extra layer of security in the chance a private key is leaked or compromised.

### Add SSH Keys
---

The preferred method is using the `ssh-copy-id` util, this assumes you already have password based SSH access to the remote server.

The utility will connect to the remote host and copy the contents of `~/.ssh/id_rsa.pub` into a file on the remote machine at `~/.ssh/authorized_keys`

```bash
ssh-copy-id username@remote_host

```

### Disable Password Based Login
---

Password based login is susceptable to brute-force style attacks so it is better off to disable password based login completely once you have established a valid login using SSH keys.

Once you have established a connection to your remote account with admin privileges, edit the SSH daemon configuration file to disable password login.

```bash
sudo nano /etc/ssh/sshd_config

```

Search for `PasswordAuthentication`, the line may be commented out so if necessary uncomment it and set the value to `no`.

Save and close the file then restart the `sshd` service.

```bash
sudo systemctl restart ssh

```

As a precaution, check that the SSH service is working properly before closing the current session. In the offchance something is incorrect at this point due to misconfiguration you risk losing access to the remote machine if you terminate your session.