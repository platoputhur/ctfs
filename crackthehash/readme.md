# tools
 - hash-identifier
 - john
 - hashid
 - hashcat

# hash #1 - 48bb6e862e54f2a795ffc4e541caed4d
 - looks like md5
 - `john --format=raw-md5 hash.txt --show`

# hash #2 - CBFDAC6008F9CAB4083784CBD1874F76618D2A97
 - looks like sha1
 - `john --format=Raw-SHA1 --wordlist=/usr/share/wordlists//seclists/Passwords/2020-200_most_used_passwords.txt hash2.txt`

# hash #3 - 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
 - looks like sha256
 - `john --format=Raw-SHA256 --wordlist=/usr/share/wordlists//seclists/Passwords/2020-200_most_used_passwords.txt hash3.txt`

# hash #4 - $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
 - no clud about the hash type, so searched for hash type in https://www.tunnelsup.com/hash-analyzer/ and it showed the hash is bcrypt. tryhackme has mentioned that the password is of length 4, so using the length argument for john as well
 - `john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt --length=4 hash4.txt`

# hash #5 - 279412f945939ba78ce0758d3fd83daa
 - looks like md5, but it is actually md4 and I couldn't get john to crack it. So used the below website to get the word
 - [md4decrypt](https://md5decrypt.net/en/Md4/#answer)

# hash #6 - F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
 - looks like sha256 and try hack me had metioned everything is there in rockyou wordlist. Also the length is 5
 - `john --format=Raw-SHA256  --length=5 --wordlist=/usr/share/wordlists/rockyou.txt hash6.txt`

# hash #7 - 1DFECA0C002AE40B8619ECF94819CC1B
 - looks like md5 but it is actally [NTLM](https://hashcat.net/wiki/doku.php?id=example_hashes) as given in the thm hint
 - `hashcat -m 1000 -a 0 a.hash rockyou.txt`
   - m 1000 - ntlm hash
   - a 0 - dictionary attack

# hash #8 - $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.
 - after looking at the [example hashes of hashcat website](https://hashcat.net/wiki/doku.php?id=example_hashes), it was not clear, so tried hash-identifier with no luck. Finally tried `hashid`
 - `hashid -m "\$6\$aReallyHardSalt\$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02."`
   - escaping of $ signs are needed since terminal considers $ as an env prefix and replaces it.
   - The above command gives us the hashcat mode 1800 as well. 
 - `hashcat -m 1800 -a 0 a.hash rockyou.txt`

# hash #9 - e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme
 - the hash file should be in the hash:salt format. when tried the hash with hashid tool, it says sha1, but we know it has a salt in it. So go through all the sha1 type algos in the examples linke from hashcat. Then we can see 160: `HMAC-SHA1 (key = $salt)` - `d89c92b4400b15c39e462a8caa939ab40c3aeeea:1234` is of the same format we have in our hash file.
 - `hashcat -m 160 -a 0 a.hash rockyou.txt`