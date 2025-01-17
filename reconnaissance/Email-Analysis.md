---
title: Email Analysis
description: Detecting malware from messages, check if they are phishing.
tags:
    - Reconnaissance
refs:
    - https://www.crowdstrike.com/cybersecurity-101/spoofing-attacks/email-spoofing/
date: 2023-10-14
draft: false
---

## OSINT

- **[InQuest Labs](https://labs.inquest.net/)**

- **[Message Header Analyzer](https://mha.azurewebsites.net/)**

    Analyses message header of email. It helps to check the phishing emails.

- **[PhishTool](https://www.phishtool.com/)**

    Combines threat intelligence, OSINT, email metadata and battle tested auto-analysis pathways into one powerful phishing response platform.

- **[Simple Email Reputation](https://emailrep.io/)**

    Paste the sender’s email address to check if the address is suspicious.

- **[VirusTotal](https://www.virustotal.com/gui/)**

<br />

## Email Source Analysis

We can see the email source in each provider or execute the following commands.

```sh
open example.xml
strings example.eml
```

Check the following headers to distinguish whether it is spoofed.

### Authentication-Results

Each value in **`dkim`, `spf`, `dmarc`** should be **`pass`**. If not, the email may be spoofed.

### Received  
    
It should match the sender’s email address.

### Received-SPF

Its value should be **`pass`**. If its value is **`faile`, `softfail`, `neutral` or `none`**, the email may be spoofed.

### Others

Besides, check if the sender displayed name or address match **`From`, `Reply-To`, `Return-Path`** in headers. However, these header values can be modified by attackers so it could not be relied perfectly.  

<br />

## Check SPF, DKIM, DMARC Records of Domain

To confirm if these records exist in a domain, use `dig` command. If not exist, the domain may be spoofed, or may be categorized spam by some email providers.  

```bash
dig example.com txt
```

Also we can check that using online tools such as [MxToolbox](https://mxtoolbox.com/).

### SPF Record

```bash
dig example.com txt

# output example:
example.com.  IN  TXT  "v=spf1 include:spf.example.com -all"
example.com.  IN  TXT  "v=spf1 +ip4:10.0.0.1/24 -all"
example.com.  IN  TXT  "v=spf1 a:mail.example.com -all"
example.com.  IN  TXT  "v=spf1 -all"
example.com.  IN  TXT  "v=spf1 mx -all"
```

### Check DKIM Record

```bash
dig selector._domainkey.example.com txt

# output examples:
selector._domainkey.example.com  IN  TXT  k=rsa;p=J8eTBu224i086iK
selector._domainkey.example.com  IN  TXT  "selector._domainkey.example.com. 0  IN  TXT  "v=DKIM1; p=ABC...123;""
```

To check `DKIM` record, we need to specify the DKIM selector. It can be found in `s=` header value of dkim section in email header such as Authentication-Results.  For example,

```bash
Authentication-Results: mx.google.com;
  dkim=pass header.i=@example.com header.s=s1234 header.b=ABCDEF123
```

In this case, we execute `dig [s1234.example.com](http://s1234.example.com) txt` .

### Check DMARC Record

```bash
dig _dmarc.example.com txt

# output examples:
_dmarc.example.com.  IN  TXT  "v=DMARC1; p=reject; rua=mailto:mailauth-reports@example.com"
```

<br />

## Be Careful of Fake Reply Message

If the email contains **`RE:`** (Regarding) which means replying your message, we need to check that it’s a really reply message. Attacker may impersonate reply message that victim communicated in the past.

<br />

## Malware Detection in Attached Files

If you got email in which attached **`suspicious`** files, you need to investigate them.

1. **View the Message Source**

2. **Copy the Attached File's Base64**

3. **Change Base64 to SHA256**

    ```sh
    sha256sum attached_file.doc
    # or
    echo -n 'abcde..==' > hash.txt
    sha256sum hash.txt
    ```

    Or there are some useful tools:

    - **[CyberChef](https://gchq.github.io/CyberChef/)** is useful to change the cipher.

4. **Investigate the Hash**

    There are some useful tools:

    - **[InQuest Labs](https://labs.inquest.net/)**

    - **[Talos File Reputation](https://www.talosintelligence.com/talos_file_reputation)**

    - **[VirusTotal](https://www.virustotal.com/gui/home/upload)**

