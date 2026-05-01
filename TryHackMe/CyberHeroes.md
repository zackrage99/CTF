# Simple Web CTF Walkthrough



---

## Recon

I started with an Nmap scan to identify open ports and running services:

```bash
nmap -sV 10.112.182.210
```


![nmap scan](https://github.com/zackrage99/CTF/blob/main/images/CyberHeroes/nmap%20scan.png)


The scan revealed two open ports:

- **22** — SSH  
- **80** — Apache web server  

Since a web server was available, I moved to browser-based enumeration.

---

## Web Enumeration

I opened the target in a browser:

```bash
http://10.112.182.210
```

The site loaded a login page.

I attempted to log in with random credentials, but it always returned a **wrong password** message. At first, this suggested a possible **XSS vulnerability**, so I tested multiple payloads — but none worked.

---

## Directory Brute Forcing

Next, I used Gobuster to look for hidden directories:

```bash
gobuster dir -u http://10.112.182.210 -w /usr/share/wordlists/seclists/Discovery/Web-content/raft-medium-files.txt
```

![gobuster](https://github.com/zackrage99/CTF/blob/main/images/CyberHeroes/gobuster.png)

After the scan completed, I found an interesting directory:

```
/assets
```

Navigating to it:

```bash
http://10.112.182.210/assets
```

---

## Asset Analysis

Inside the `/assets` directory, I explored the folders and found a **JavaScript file**.

Reviewing it revealed a version of **Bootstrap (3.7.0)**. I searched for known vulnerabilities related to that version, but nothing useful came up.

At this point, I shifted focus back to the main page.

---

![javascript](https://github.com/zackrage99/CTF/blob/main/images/CyberHeroes/js.png)

## Source Code Analysis

Back on the login page, I viewed the page source (**Right Click → View Page Source**) and found a critical piece of JavaScript:

![page source](https://github.com/zackrage99/CTF/blob/main/images/CyberHeroes/page%20source.png)
```javascript
if (a.value=="h3ck3rBoi" & b.value==RevereString("54321@terceSrepuS"))
```

This revealed:

- **Username:** `h3ck3rBoi`  
- **Password:** A reversed string of `54321@terceSrepuS`

---

## Password Recovery

To reverse the string, I used the `rev` command in Linux:

```bash
echo "54321@terceSrepuS" | rev
```

This produced the correct password:

```
SuperSecret@12345
```

---

## Login & Flag

Using the discovered credentials:

- **Username:** `h3ck3rBoi`  
- **Password:** `SuperSecret@12345`  

I logged into the application successfully and obtained the flag.

---

