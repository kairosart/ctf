## Nmap 
### Ports and OS
```
sudo nmap -O <MACHIINE IP>
```

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-02 14:04 EDT
Nmap scan report for 10.10.183.153
Host is up (0.048s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE  SERVICE
<font color="#ffc000">22/tcp   open   ssh</font>

<font color="#ffc000">80/tcp   open   http</font>

<font color="#ffc000">8080/tcp closed http-proxy</font>

Aggressive OS guesses: Linux 3.10 - 3.13 (89%), Linux 5.4 (89%), Linux 3.10 - 4.11 (88%), Linux 3.12 (88%), Linux 3.13 (88%), Linux 3.13 or 4.2 (88%), Linux 3.2 - 3.5 (88%), Linux 3.2 - 3.8 (88%), Linux 4.2 (88%), Linux 4.4 (88%)
No exact OS matches for host (test conditions non-ideal).

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.09 seconds

### Discover interesting paths/folders

```
nmap --script http-enum -sV <MACHIINE IP>
```
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)

80/tcp   open   http       nginx 1.14.0 (Ubuntu)

|_http-server-header: nginx/1.14.0 (Ubuntu)
| http-enum: 
|   /info.php: Possible information file
<font color="#ffc000">|_  /.git/HEAD: Git folder</font>
[[GitHack]]

8080/tcp closed http-proxy
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 131.49 seconds

