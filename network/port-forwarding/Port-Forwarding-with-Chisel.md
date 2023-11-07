---
title: Port Forwarding with Chisel
description: Chisel is a fast TCP/UDP tunnel over HTTP. Is can be used for port forwarding.
tags:
    - Network
refs:
    - https://github.com/jpillora/chisel
date: 2023-05-20
draft: false
---

## Transfer Chisel Binary to Remote Machine

If the remote machine does not have chisel binary, we need to transfer it from local machine (if local machine has the binary).

```bash
# In local machine
python3 -m http.server --directory /path/to/chisel/directory

# In remote machine
wget http://<local-ip>:8000/chisel
chmod +x chisel
./chisel -h
```

<br />

## Port Forwarding

```sh
# In remote machine
chisel server -p <listen-port>

# In local machine
chisel client <listen-ip>:<listen-port> <local-port>:<target-ip>:<target-port>
```

<br />

## Reverse Port Forwarding

It is useful when we want to access to the host & the port that cannot be directly accessible from local machine.

```sh
# In local machine
chisel server -p 9999 --reverse

# In remote machine
# replace 10.0.0.1 with your local ip
chisel client 10.0.0.1:9999 R:8090:172.16.22.2:8000
```

After that, we can access to **`http://localhost:8090/`** in local machine. In short, we can access to **`http://172.16.22.2:8000/`** via **`localhost:8090`**.  
Try **`curl`** to confirm.

```sh
curl http://localhost:8090

# The result is the content of http://172.16.22.2:8000/
```

### Example (SSH)

Assume we want to connect to SSH server (**`ssh://172.17.0.1:22`**) that cannot be directly accessed from local machine.  
 
```sh
# In local machine
chisel server -p 9999 --reverse

# In remote machine (assume we want to connect ssh://172.17.0.1:22)
chisel client <local-ip>:9999 R:2222:172.17.0.1:22
```

After that, we can connect to the SSH server from local machine.  
Run the following command in local machine.

```bash
ssh user@localhost -p 2222
```

### Forward Multiple Ports

```bash
# In local machine
chisel server -p 9999 --reverse

# In remote machine
chisel client 10.0.0.1:9999 R:3000:127.0.0.1:3000 R:8000:127.0.0.1:8000
```

After that, we can access to **`http://localhost:3000`** and **`http://localhost:8000`** in local machine.

<br />

## Forward Dynamic SOCKS Proxy

```sh
# In remote
chisel server -p 9999 --socks5

# In local
chisel client 10.0.0.1:9999 8000:socks
```

Then modify **`/etc/proxychains.conf`** in local machine.  
Comment out the line of **"socks4"**.

```sh
# /etc/proxychains.conf
...
socks5 127.0.0.1  8000
```

<br />

## Reverse Dynamic SOCKS Proxy

It is useful when we want to access to the host & multiple ports that cannot be directly accessible from local machine.

```bash
# In local machine
chisel server -p 9999 --reverse

# In remote machine
chisel client 10.0.0.1:9999 R:9000:socks
```

Then modify **`/etc/proxychains.conf`** in local machine.  
Comment out the line of **"socks4"**.

```bash
# /etc/proxychains.conf
...
socks5 127.0.0.1 9000
```

To confirm if we can reach the desired host and port, run **nmap** with **proxychains**.

```bash
proxychains nmap localhost
```

### Enable Proxychains Bash

It allows us to execute programs without adding **proxychains** command before main command.

```bash
proxychains bash

# Run some command without "proxychains" command.
nmap localhost
```

<br />

### Burp Suite Settings for Proxy

If we want to use **Burp Suite** with **proxychains**, we can add the **SOCKS** proxy in the Proxy settings.  
For details, please see the [SOCKS Proxy in Burp Suite](/exploit/web/tool/socks-proxy-in-burp-suite).