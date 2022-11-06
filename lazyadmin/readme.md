export ip=10.10.44.83

# recon
## tool - nmap
- `nmap -sC -sV -oG nmap.log $ip`
  - 80, 22 open

## tool - nikto
- `nikto -h $ip | tee nikto.log`

## tool - gobuster
- `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip | tee gobuster.log`
- `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip/content | tee gobuster.log`

# process
- gobuster showed an interesting directory called `content`, so ran gobuster on `$ip/content` which gave a changelog with the CMS name and version installed under the `content` dir.
- Found another interesting dir `inc`. inside that we can see many interesting files including the mysql database backup.
- `as` is the admin panel
- the mysql backup file has the username and password's md5 hash using which we are able to login
- upon checking the available exploits for the installed version of the cms, we see we atleast one type of [exploit](https://www.exploit-db.com/exploits/40700) which we will use to gain access to the server
- We go to the ads menu in the panel and add the content of a php reverse shell in that ads body section and give some random name. Use this random name to access reverseshell file $ip/content/inc/ads/randomname.php. Make sure you have already set up then nc using `nc -lnvp 4444` and updated the php reverse shell contents to include the host systems ip and port.
- As soon as you access you should gain access to the server via terminal.
- listing the contents of /home/itguy you see there are many files, one among them is a mysql user:password combo and the other is the answer to the first question.
- login to mysql using `mysql -u rice -p` and enter the password you got from the file.
- now when you look at the `sudo -l` command you see `(ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl` which mean `www-data` the user you are currently logged in as in reverse shell, can run the above command as sudo.
- we have the permission to read backup.pl in the home dir /home/itguy and opening backup.pl we see it is actually executing another file called `copy.sh` in `/etc`. if we check this file's permissions we see, we can actually edit this file.
- So we will edit to add a bash reverse shell with our ip and port. So we start our nc listener and then run the sudo command we got from `sudo -l`
  - `sudo /usr/bin/perl /home/itguy/backup.pl`
- Now we have our root shell, and in the `/root` we have the final answer for finishing this ctf

