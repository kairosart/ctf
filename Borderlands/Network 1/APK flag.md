API key that fits the following pattern "AND*"
## Reverse engineer the apk using apktool

`$ apktool d mobile-app-prototype.apk`

Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
I: Using Apktool 2.7.0-dirty on mobile-app-prototype.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /home/kali/.local/share/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...

`$ cd mobile-app-prototype`

## Make a search for the variable name
`$ grep -rn 'encrypted_api_key'`

name="encrypted_api_key">**CBQOSTEFZNL5U8LJB2hhBTDvQi2zQo**

## Take a look at the api.php file
if (!isset(_GET['apikey']) || ((substr(GET'apikey', 0, 20) !== "WEBLhvOJAH8d50Z4y5G5") && substr(_GET['apikey'], 0, 20) !== "**ANDVOWLDLAS5Q8OQZ2tu**" && substr(_GET'apikey', 0, 20) !== "GITtFi80llzs4TxqMWtC"))

### Comparison between the two strings

CBQOSTEFZNL5U8LJB2hhBTDvQi2zQo (Encrypted)
ANDVOWLDLAS5Q8OQZ2tu                     (Plaintext)

You will notice the numbering is in the same position. So, this a vigenere cipher.

## Vigenere Cipher
If you refer back to the jadx, you will see the decrypt function require a key. To obtain the key, simply do a reverse key search based on the vigenere table.

![[vineger1.png]]

The left letter bar represents plaintext, the upper letter bar represents key (either one, the table is symmetrical) and the inner table is ciphertext. Let’s take the second letter and compare it.

```
B (Encrypt)
N (Plaintext)
O (Key)
```

![[vineger2.png]]

Keep repeating the process, you will eventually find the key. The key is CONTEXT.

### Get the API Key
You can get the API key https://gchq.github.io/CyberChef/#recipe=Vigen%C3%A8re_Decode('CONTEXT')&input=Q0JRT1NURUZaTkw1VThMSkIyaGhCVER2UWkyelFv.

Choose the "Vigenere Decode" option.
Enter the Encrypt string and the Key.

![[apikey.png]]

API key that fits the following pattern "AND*" is **ANDVOWLDLAS5Q8OQZ2tuIPGcOu2mXk**