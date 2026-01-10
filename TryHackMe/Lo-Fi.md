LOFI Room Walkthrough - TryHackMe

### Initial Reconnaissance
Nmap Scan

First, after launching the machine, I performed an Nmap scan to identify open ports and services:
bash

nmap -sV -nP 10.48.141.113

Results:

    Port 80 (HTTP) - Web server

    Port 22 (SSH) - SSH service (not explored further)

### Web Enumeration

Navigated to the web application at http://10.48.141.113

The website appeared to be a music streaming service with various sections. After inspecting the webpage, I found a "Discovery" section with links to different music categories.

***Vulnerability Discovery***
After spending a whiile inspecting the website didn't find anything interesting


Parameter Testing

Noticed that the website used a page parameter in the URL:

    http://10.48.141.113/?page=relax.php

This suggested potential for Local File Inclusion (LFI) or directory traversal vulnerabilities.
Directory Traversal Attempt

Tested for directory traversal vulnerability:
text

http://10.48.141.113/?page=../../../../etc/passwd

Success! The server returned the contents of /etc/passwd, confirming the vulnerability.
Flag Extraction
### Finding the Flag File

Assuming the flag would be in the root directory or a standard location, I attempted:
text

http://10.48.141.113/?page=../../../../flag.txt

Result: Successfully retrieved the flag contents!
