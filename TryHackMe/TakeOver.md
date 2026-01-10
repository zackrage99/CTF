FutureVera CTF Walkthrough 🚀
***URL:*** https://tryhackme.com/room/takeover
1. Initial Scanning 🔍

The first step is always to identify open ports and services using nmap:
Bash

nmap -sV -Pn 10.10.XX.XX

Results:

    Port 22: SSH

    Port 80: HTTP

    Port 443: HTTPS

The room description provides the domain: https://futurevera.thm. I mapped this IP to the domain in my hosts file:
Bash

sudo nano /etc/hosts
# Add: 10.10.XX.XX futurevera.thm

2. Website Inspection 🌐

After opening https://futurevera.thm, the site didn't reveal much. However, a hint in the description mentioned, "we are rebuilding our support." This strongly suggests a support subdomain exists.
3. Subdomain Hunting with WFuzz 🏹

I used wfuzz to scan for hidden subdomains. My first attempt returned too many 200 OK responses, so I had to filter the results:
Bash

# Filtered scan to hide the noise
wfuzz -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.futurevera.thm" --hc 200 https://futurevera.thm

Found 2 subdomains:

    blog.futurevera.thm

    support.futurevera.thm

I updated /etc/hosts again:
Plaintext

10.10.XX.XX blog.futurevera.thm
10.10.XX.XX support.futurevera.thm

4. The Certificate Trick 🎟️

While blog was a dead end, support.futurevera.thm looked empty. I decided to inspect the SSL Certificate details (click the padlock icon in the browser).

Under the Subject Alternative Name (SAN), I found a hidden, long-form subdomain:

    DNS Name: ******desk934752.support.futurevera.thm

I added this final domain to /etc/hosts:
Bash

10.10.XX.XX ******desk934752.support.futurevera.thm

5. The "Aha!" Moment 💡

Visiting the new subdomain via HTTPS resulted in a blank page. After some trial and error, I tried accessing it via Port 80 (HTTP):

http://******desk934752.support.futurevera.thm:80

The page failed to load, but the error message leaked everything:
Plaintext

Hmm. We’re having trouble finding that site.
We can’t connect to the server at flag{***********}.s3-website-us-west-3.amazonaws.com.

The subdomain was pointing to an AWS S3 bucket, and the bucket name itself was the flag!

Flag: flag{***********} 🎉
