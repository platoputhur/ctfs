export ip=10.10.37.45


# Recon and Scanning

## tool - nmap
- `nmap -sC -sV -O -oG nmap.log $ip`
  - 80 & 22 are open

## tool - gobuster
- `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip | tee gobuster.log`
  - got a peculiar dirs `r`, `poem`, so running gobuster on these dir again
  - got another dir `a`, so doing another gobuster/
  - now we got `b`. This made me thinking could it be, `r\a\b\b\i\t`, looks like I was right, and in the webpage source we can see what looks like a username:password combo.
  - Considering we have ssh open, tried to login using this credentials successfully.
  - execute linpeas.sh to do basic enumeration and we see perl has capabilities set: `/usr/bin/perl = cap_setuid+ep`. This can be used to do privilege escalation
  - looking for any SUID set binaries, didn't find anything useful
  - checked for the permissions/ownerships of the files we have on the home dir, there is one we can read but owned by root.
  - checking for all the commands alice is allowed run using sudo, and we see python3.6 can be run as the user rabbit with the file path for the py file from the home dir
  - We can see the py file in the home dir which just prints out some poem, but it is importing random package. So if we create a `random.py` file with whatever we want, executing the poem printing py file will also execute the random.py contents.
  - so adding a random.py file in the home dir with the below contents
    - `import os`
    - `os.system("/bin/bash")`
  - executing `sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py` give us rabbit's shell
  - in the home dir of rabbit, we can see there is a file with SUID set. Running it just crashes the file, so tried opening it with a text editor and we see, it is using `date` binary without the absolute path.
  - considering the absolute path is not given, we can create a `date` file and then add `/bin/bash` to it. and then change the PATH env to include the dir where we have created the `date` file. date file content can be like below, also run `chmod +x date` so that the file is an executable
    - `#!/bin/bash`
    - `/bin/bash`
  - Once this is done, we can execute `teaParty` to get the shell of the `hatter`
  - after this, we can see the password of `hatter` is given in the home dir of `hatter`
  - log in to ssh using username hatter and given password.
  - Then we know we have the perl's capabilities set from the linpeas enumeration log. So we can use that to [get the root shell](https://gtfobins.github.io/gtfobins/perl/#capabilities)
    - `perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'`
    - now we can get the root flag from the alice users home dir



