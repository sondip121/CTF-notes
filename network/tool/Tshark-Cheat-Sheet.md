---
title: Tshark Cheat Sheet
description: Tshark is a terminal-oriented version of Wireshark. It's a network protocol analyzer.
tags:
    - Network
refs:
date: 2023-10-26
draft: false
---

## Basic Capture

```sh
tshark

# -i: interface (default: eth0)
tshark -i tun0
# --list-interface: List interfaces available
tshark --list-interfaces

# -r: Use a captured file
tshark -r example.pcapng

# Number of packets
tshark -r example.pcapng | wc -l
```

<br />

## Filtering

We can filter packets using `-Y` option.

### Protocols

```bash
# HTTP
tshark -Y 'http'

# ICMP
tshark -Y 'icmp'

# TCP/UDP
tshark -Y 'tcp'
tshark -Y 'udp'
```

### IP Address

```bash
tshark -Y 'ip.addr == 127.0.0.1'

# Source address
tshark -Y 'ip.src == 127.0.0.1'

# Destination address
tshark -Y 'ip.dst == 127.0.0.1'
```
