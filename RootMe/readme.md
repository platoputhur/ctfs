export ip=10.10.181.248

# tool - nmap: 
 - nmap -sV -sC -oN nmap/initial $ip

# open ports
 - 20
 - 80

# tool - gobuster
 - gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip
    - detected files which includes index.php panel/, uploads/
    - panel is a hidden page and it has a file upload form
    - any file uploaded from /panel is directly going into /uploads/ dir and is accessible by giving the url ip/uploads/filename.ext
    - uploading a reverse shell here, would let us access that file, considering this is a php website, a php reverse shell might get us into the box
    - php files are forbidden, so renamed the file to use .phar to bypass the file ext blacklist
    - running nc to listen to the reverse shell - `nc -v -n -l -p 1234`
    - current user is www-data, used whoami command
    - traversed to various directories and all the users with home dirs are inaccessible
    - find the files with SUID set. If SUID is set, that means, these files can be run with the permissions of the owners of these files.
    - Used the command `find / -perm -u=s -type f 2>/dev/null` to get the list of files with SUID set. `/usr/bin/python` has SUID set
    - So we can use this python to get elevated privileges. gtfobins has python one liner to get this.
        - `python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`
    - Checking whoami to see if we have root, and then going to /root, we got the final clue.