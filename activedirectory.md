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
Now configure ntlmrelayx.py (locate ntlmrelay) python3 ntlmrelayx.py -tf targets.txt -smb2support <br>


