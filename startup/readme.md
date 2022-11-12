export ip=10.10.231.170

# Recon and scanning

## nmap
- `sudo nmap -sV -sC -oG nmap.log $ip`
  - 21, 22, 80 open ports
  - it also showed anonymous ftp dir `ftp` is writable.
## gobuster
- `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip | tee gobuster.log`
  - showed one dir `files` which has just one image, one txt file and one empty ftp directory.
  - the text file revealed a users name: `maya`

# process
- uploading a php shell using ftp as anonymous file write is permitted
- starting a shell using the command `nc -lnvp 4444`
- got the session after going to the url of the shell we uplaoded viw ftp
- looked around in the shell, and found a file called `recipe.txt` at `\` which contained the first flag
- then looked around and found there is a user called `lennie` but the home dir was inaccessible for the current user.
- after a bit more looking around found one director called `incidents` at `\` with read permissions for `www-data` which is the current user
- it had file called `suspicious.pcapng` which seemed like wireshark packet capture file.
- so opened it using a file which can show the contents in human readable format. one tool is wireshark itself, but I used the below command frpm kali.
  - `tcpick -C -yP -r suspicious.pcapng`
- here in this file I was able to find failed login attempts with a password. So I took that password and tried sshing into the machine using the user lennie. and indeed this password worked.
- in the home dir of this user, we have our next flag.
- looking around we can see there are other directories in the home dir of lennie, there is dir called `scripts` which has one `.sh` file.
- this file is owned by root and this file calls another file which is writable by the current user. So I started a listener in my host machine and added a reverse shell command inside that `/etc/print.sh` file. I went ahead and checked crontab lists, but it said no jobs for the user `lennie`. So used `crontab -l -u root` to view any cron jobs for root, but the current user was not privileged to see root's crontabs. By the time I got the reverseshell connected and the user was root. So that was a bit of luck I guess. My next plan was to check for any tools which lets us see the all the cronjobs irrespective of the users. Anyway now that I have root, I checked the home dir of root, and there was our final flag.