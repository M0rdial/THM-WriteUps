## Room Information

Platform: TryHackMe

Room Name: Vulnversity

Difficulty: [Easy]

Date Completed: [03/03/2026]

## Objective

The objective of this room was to perform enumeration, identify vulnerabilities, gain initial access, and escalate privileges to root.

## Summary

Active Recon, Web app attacks and Privilege escalation:
 - Gather information about target machine using a network scanner tool
 - Use a directory discovery tool
 - Upload and execute our payload, which leads to compromising the web server

## Tools Used
 - Rustscan/Nmap
 - Enum4linux
 - Gobuster
 - BurpSuite

## Reconnaissance

### Summary

We use the rustscan tool to know which ports are open and which service they are running and their versions.

Scan for open ports:

```
❯ rustscan -a 10.113.142.126 -- -sV                                                                     
[~] File limit higher than batch size. Can increase speed by increasing batch size '-b 4900'.           
Open 10.113.142.126:21                                                                                  
Open 10.113.142.126:22                                                                                  
Open 10.113.142.126:139   
Open 10.113.142.126:445   
Open 10.113.142.126:3128  
Open 10.113.142.126:3333  
[~] Starting Nmap         
[>] The Nmap command to be run is nmap -sV -vvv -p 21,22,139,445,3128,3333 10.113.142.126               

Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-03 21:54 +0200                                       
NSE: Loaded 48 scripts for scanning.                
Initiating Ping Scan at 21:54                       
Scanning 10.113.142.126 [4 ports]                   
Completed Ping Scan at 21:54, 0.24s elapsed (1 total hosts)                                             
Initiating Parallel DNS resolution of 1 host. at 21:54                                                  
Completed Parallel DNS resolution of 1 host. at 21:54, 0.50s elapsed                                    
DNS resolution of 1 IPs took 0.50s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]        
Initiating SYN Stealth Scan at 21:54                
Scanning 10.113.142.126 [6 ports]                   
Discovered open port 445/tcp on 10.113.142.126      
Discovered open port 139/tcp on 10.113.142.126      
Discovered open port 3128/tcp on 10.113.142.126     
Discovered open port 3333/tcp on 10.113.142.126     
Discovered open port 22/tcp on 10.113.142.126       
Discovered open port 21/tcp on 10.113.142.126       
Completed SYN Stealth Scan at 21:54, 0.25s elapsed (6 total ports)                                      
Initiating Service scan at 21:54                    
Scanning 6 services on 10.113.142.126               
Completed Service scan at 21:54, 22.58s elapsed (6 services on 1 host)                                  
NSE: Script scanning 10.113.142.126.                
NSE: Starting runlevel 1 (of 2) scan.               
Initiating NSE at 21:54   
Completed NSE at 21:54, 0.96s elapsed               
NSE: Starting runlevel 2 (of 2) scan.               
Initiating NSE at 21:54   
Completed NSE at 21:54, 0.91s elapsed               
Nmap scan report for 10.113.142.126                 
Host is up, received reset ttl 62 (0.22s latency).
Scanned at 2026-03-03 21:54:33 SAST for 25s         

PORT     STATE SERVICE     REASON         VERSION   
21/tcp   open  ftp         syn-ack ttl 62 vsftpd 3.0.5                                                  
22/tcp   open  ssh         syn-ack ttl 62 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
139/tcp  open  netbios-ssn syn-ack ttl 62 Samba smbd 4                                                  
445/tcp  open  netbios-ssn syn-ack ttl 62 Samba smbd 4                                                  
3128/tcp open  http-proxy  syn-ack ttl 62 Squid http proxy 4.10                                         
3333/tcp open  http        syn-ack ttl 62 Apache httpd 2.4.41 ((Ubuntu))                                
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel                                          

Read data files from: /usr/share/nmap

```

Results
 Open Ports, info and notes:
  - On port 21 the service running is ftp and version is vsftpd 3.0.5, anonymous login possible or exploit available.
  - On port 22 the service running is ssh and version is OpenSSH 8.2p1
  - On port 139 and 445 the service running is netbios-ssn and version is Samba smbd 4
  - On port 3182 the service running is an http-proxy and the version is Squid http proxy 4.10
  - On port 3333 the service running is http(webserver) and the version is Apache httpd 2.4.41, web enumeration required and web exploitation possible.

Answering the questions:

- **Scan the box; how many ports are open?** 6 ports
 
- **What version of the squid proxy is running on the machine?** 4.10
 
- **How many ports will Nmap scan if the flag -p-400 was used?** 400

- **What is the most likely operating system this machine is running?** Ubuntu

- **What port is the web server running on?** 3333

- **What is the flag for enabling verbose mode using Nmap?** -v

## Website Enumeration

### Summary

We enumerate the website through brute-forcing to any hidden directories that we can use to compromise the web server.

Enumerate for directory:
 
```

❯ gobuster dir -u http://10.113.142.126:3333 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 357 -ne
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.113.142.126:3333
[+] Method:                  GET
[+] Threads:                 357
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-1.0.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
images               (Status: 301) [Size: 324] [--> http://10.113.142.126:3333/images/]
css                  (Status: 301) [Size: 321] [--> http://10.113.142.126:3333/css/]
js                   (Status: 301) [Size: 320] [--> http://10.113.142.126:3333/js/]
internal             (Status: 301) [Size: 326] [--> http://10.113.142.126:3333/internal/]
Progress: 141707 / 141707 (100.00%)
===============================================================
Finished
===============================================================

```

Results:
 Directories:
  - /images
  - /css
  - /js
  - /internal


* After manual investigating the the directories that I found, /internal has file upload form page. It is possible to get reverse shell by uploading a payload.
<img src="https://imgur.com/gallery/file-upload-form-page-XnytYvc">
 
