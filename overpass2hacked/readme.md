# Write up for the tryhackme ctf: [overpass2hacked](https://tryhackme.com/room/overpass2hacked)

- open wireshark and load the pcap file
- considering the question asking for a url for reverse shell upload, I thought of searching for the string related to revershell and hacking. 
- I searched for the string `payload` which lead to the answer for the first flag
- once you find the post req in the packet info, open it and copy the content as text to get the revere shell payload contents and that's the second flag.
- the next question is asking for password. so in linux you would need password whenever you use `sudo`, so search for the string `sudo` and analyse the next few packets to get the answer for the next flag.
- next we need to see what the user used to make the shell persistent. if you go through a few more packets, you will see the user attempting to clone a repository and you have the next flag.
- now the next question is asking how many passwords were the user able to crack. the cracking would have done on the attackers system, so there is no use of searching for the passwords in the packet file. Instead we should search for the hash file which the user would have had to at least display to copy it to the his/her system. and if you had analysed the packet file at the beginning you would have noticed a packet with the word shadow in it. considering linux stores it's user password hashes in shadow file, we can try getting the hashes and then try to crack it ourself using the wordlist `fasttrack` as mentioned in the flag's question
- I can just follow the tcp stream in wireshark to get the shadow file contents. So I right clicked on the packet where the user tried use `cat /etc/shadow` to view the contents and copied the hashes to crack it using john or hashcat. But then, the flag is the number of passwords cracked, and the hashes are available for only 5 users. so I thought of trying 5, 4, 3, 2, 1 and I got the flag.
- next we need to look at the source code of the backdoor used by the hacker. the hash and salt is ready available from the source code if you just `ctrl+f` and search for it. now we got the next 2 flags
- then we need to get the hash used by the user to pass to the backdoor. For this just go back to the tcp stream view in wireshark and scroll down until you see the hash. hint: check after the user clones the github repo of the backdoor. and this gives us the next flag
- now we need to crack the hash to get the password used by the attacker. so the backdoor adds a salt for the password, now we need to use this along with the hash to crack the password. before that we need to find the hash format. this is quickly verifiable using `hashid -m <hash>`. use this format to get the format which is sha512
- now considering we need to use the salt, go to [hashcat examples](https://hashcat.net/wiki/doku.php?id=example_hashes) and see how we can give salt along with sha512 so that hashcat can crack it.
  - I used format `1710` and appended the salt with the hash. format eg: `hash:salt` 
  - then all I needed to do to get the hash cracked was to execute the below command
    - `hashcat -m 1710 -a 0 -o pass.txt hash.txt rockyou.txt`
  - and we got the next flag.

```
export ip=10.10.245.233
export hip=10.18.25.169
```

- opening the ip address gets you the first flag
- so we are supposed to use the information we gained using the analysis, that means the easiest way in would probably be using the backdoor the hacker created
- first of all we need to find the port the backdoor uses. going to the github page for the backdoor we can see it is using port `2222`. 
- the user in the system can be easily viewed from the wireshark tcp stream we had opened. we already have cracked the backdoor password.
- using all this information we login using ssh at port `2222`, but it is using the older encryption `ssh-rsa`.
  - `ssh james@10.10.245.233 -p2222 -oHostKeyAlgorithms=+ssh-rsa`
- at the users home dir we get the next flag.
- next I ran the below command to see if there are any executables with SUID set
  - `find / -perm -u=s -type f 2>/dev/null`
- and there was a file with the name `.suid_bash`, so I tried suid_bash --help to see if it outputs and help text and I was not disappointed. It was actually a bash binary with SUID set literally lol ;)
- upon seeing this, immediately I went to [gtfobins](https://gtfobins.github.io/gtfobins/bash/#suid) to see what is the bash argument to get the shell as I couldn't remember it
  - `./.suid_bash -p` gets the root shell and we can go to the root dir to get the final flag.
