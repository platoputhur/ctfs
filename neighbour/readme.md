# Write up for the tryhackme ctf: [neighbour](https://tryhackme.com/room/neighbour)

```
export ip=10.10.203.217
export hip=10.18.25.169
```

## recon and scanning
### nmap
- `sudo nmap -sC -sV -oN nmap.log -O $ip`


## process
- from the nmap scan we know http server is running
- accessing the http server shows a login page.
- checking the source of the page shows guest login credentials. Also shows the possibility of `admin` username.
- logging into the webapp using the guest credentials shows the welcome page
- looking at the url of the welcome page we can see it has query param `guest` which is the current username
- now trying the same with `admin` as the username
- this gives the flag.