# Futurevera - TryHackMe Walkthrough

## Reconnaissance

### Port Scanning
```bash
nmap -sV -nP 10.10.XX.XX

Results:

    Port 22: SSH

    Port 80: HTTP

    Port 443: HTTPS

Domain Setup

From description: futurevera.thm

Add to /etc/hosts:
bash

sudo nano /etc/hosts
# Add: 10.10.XX.XX futurevera.thm

Subdomain Enumeration
WFuzz Scan
bash

wfuzz -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.futurevera.thm" https://futurevera.thm

Too many 200 responses → Filter them:
bash

wfuzz -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.futurevera.thm" --hc 200 https://futurevera.thm

Discovered:

    blog.futurevera.thm

    support.futurevera.thm

Update /etc/hosts:
bash

10.10.XX.XX blog.futurevera.thm
10.10.XX.XX support.futurevera.thm

Certificate Inspection
Hidden Subdomain Discovery

Check SSL certificate for support.futurevera.thm:

    Click padlock icon → Certificate

    Check "Subject Alternative Name" field

Found: ******desk934752.support.futurevera.thm

Add to /etc/hosts:
bash

10.10.XX.XX ******desk934752.support.futurevera.thm

The Flag
Error Message Analysis

Accessing the subdomain showed nothing. Try port 80:
text

http://******desk934752.support.futurevera.thm:80

Error message contained:
text

Hmm. Weâ€™re having trouble finding that site.
We canâ€™t connect to the server at flag{***********}.s3-website-us-west-3.amazonaws.com.

🏁 Flag

flag{***********}
