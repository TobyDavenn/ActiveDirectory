Identify network 

NMAP Scan with -sC -A

Look over all services and any common vulnerabilities for the versions

Web

Look over all web servers, brute force directories and look for virtual hosts (anything revealing potential virtual hosts? Add to /etc/hosts file

Look for software versions and check online for CVEs

Can use searchsploit aswell

Download POCs and run

Brute force subdomains

SMB
Look over nmap findings

Use smb scanners on msfconsole

Run smb_version on msfconsole and make note - search web for cves

Check null shares - Use smbclient -L \\\\IP\\
Connect to fileshare with smbclient \\\\IP\\fileshare


Run Nessus

Run Nuclei
![image](https://user-images.githubusercontent.com/35967437/200789372-f622f2e0-21cc-41f0-969f-e28e017a29a3.png)
