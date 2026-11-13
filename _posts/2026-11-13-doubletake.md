---
title: DoubleTake - Hack The Box Walkthrough
date: 2026-11-13 00:00:00 +/-0530
categories: [Hack The Box Writeups, Easy]
tags: [cve-2026-123158,ssh,cve-2026-41567,docker,linux] 
media_subpath: /assets/img/doubletake/
image: img.png
---
DoubleTake is a CVE focused machine, what I mean by that is it does not get exploited by normal exploitaion techniques like you'd expect in a web app. We start with a CVE and go forward with it to get a RCE then exploit another CVE to get higher priv. I know all exploitations are registered as some CVE but these are of the type you'll learn when you read the CVE for that specific exploit.

Let's start.

## Enumeration
### nmap
We start wth a nmap scan as always
```terminal
└─$ nmap 10.129.55.79
                
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
so port 22, 80 are open, we'll run a full scan for some additional info

```terminal
└─$ nmap 10.129.55.79 -A -p80,22

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.6 (protocol 2.0)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
| http-auth: 
| HTTP/1.1 401 UNAUTHORIZED\x0D
|_  Basic realm=Zentara
```
If we visit the port 80, we get prompted with a login form, so to access it we'll have to get the credentials first from somewhere, and that leaves the ssh but we dont have the ssh credentials as well, hmmm you chose this life bro dont blame the machine. We will look for exploits using the info we got from the nmap scan.

## Vulnerabilty Assesment
Okay so it is running OpenSSH 10.6 (protocol 2.0), now after some research I found out this specific version of OpenSSH has a [CVE-2026-123158](https://www.ox.security/blog/openssh-authentication-bypass-cve-2026-123158/) aka *sshole (yes). It was very messy to understand yet it sounded so simple (it is very simple), thanks to ozgun kultekin for his amazing website [here](https://sshhole.com/), he explained it wayy easy and in the end you can perform and see too how easy it is to crack it.

![](sshole.png){: .shadow w="800"}

## Initial Foothold
It is really very easy, all you have to do is for the ssh as the alpine user, enter a pass (whatever you want) then enter another password >256 bytes and you are good to go

![](alpinessh.png){: .shadow w="800"}

and just like that you get a alpine shell. now lets try to cat the user flag

![](flagfail.png){: .shadow w="800"}

looks like we don't have the permission, no problem them let's get the login credentials then of the web app. If you go the alpine user and list all the files you'll find a mozilla file and inside it there will be credentials for the user bob. **Both of them are base64 encoded.**

![](mozilla.png){: .shadow w="800"}

```
bob:CorrectHorseBatteryStaple
```
nicee, now before going to the web app, lets try getting a ssh for bob using the credentials we got. If we suceed we can pwn the user. 

![](user-flag.png){: .shadow w="800"}

okay **user pwned**

## Privilege Escalation 