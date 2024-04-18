**Hint:** Explore the APK.

Visit the Web Site http://10.10.X.X.

![[context_web.png]]
Download the apk file from [here](http://10.10.65.175/mobile-app-prototype.apk).

Install JADX too in Kali Linux.
	`sudo apt install jadx`

Run JADX.
	`jadx-gui`

Open the apk file you downloaded.![[jadx1.png]]

The encrypted key variable is called **‘encrypted_api_key’**.

Reverse engineer the apk using apktool.
```
$ apktool d mobile-app-prototype.apk
$ cd mobile-app-prototype
$ grep -rn 'encrypted_api_key'
```

<public type="string" name="encrypted_api_key" id="0x7f0b0028" />
public type="string" name="encrypted_api_key" id="0x7f0b0028"
name="encrypted_api_key">**CBQOSTEFZNL5U8LJB2hhBTDvQi2zQo**

## The WEB flag
You have to check out every file to find the WEB flag, but you can use this command to make it easier:

`$ grep -r "WEB" 10.10.25.126` 

On /home.php you'll find apikey=**WEBLhvOJAH8d50Z4y5G5g4McG1GMGD**.