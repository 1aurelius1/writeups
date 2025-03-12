# Brute It
Brute It is a TryHackMe room focused on brute-forcing, hash cracking, and privilege escalation.

I first join the room, launch the target, and receive a target IP of 10.10.221.119.

## Recon
![](nmap-scan.png)
We can start a nmap scan using the flags `-sC` and `-sV` to get the service, version, and any additional information about the IP. We find two services open: OpenSSH running on port 22 and an Apache server running on port 80.

I'll visit the Apache server first to see if anything interesting is going on.
![](ubuntu-page.png)
Nope, nothing interesting going on. Just the default Apache Ubuntu page.

Since we have no more information to run off of, I started enumerating web pages with Gobuster to see if I could find a foothold.
![](gobuster-enum.png)
There we go, an admin login form!
![](login-page.png)

## Brute Forcing
Let's boot up Burp Suite and capture the login POST request.
![](post-request.png)
We can use this POST request in Hydra to brute force the login HTML post form.
![](hydra-brute.png)
I guess that the username is probably admin, considering it's an admin login form. We'll use rockyou.txt as our wordlist and use 'Username or password invalid' as our failure criteria. After only a couple of seconds, we get our login credentials.

We can use these to login to the admin page. We're greeted by one of our flags and a private RSA key. How fun!
![](login_success.png)

We can use this private RSA key in conjunction with ssh2john to generate a hash that we can crack later. First, let's echo the private key into a new id_rsa file. We can do this with:

<code>echo '-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,E32C44CDC29375458A02E94F94B280EA

JCPsentybdCSx8QMOcWKnIAsnIRETjZjz6ALJkX3nKSI4t40y8WfWfkBiDqvxLIm
UrFu3+/UCmXwceW6uJ7Z5CpqMFpUQN8oGUxcmOdPA88bpEBmUH/vD2K/Z+Kg0vY0
BvbTz3VEcpXJygto9WRg3M9XSVsmsxpaAEl4XBN8EmlKAkR+FLj21qbzPzN8Y7bK
HYQ0L43jIulNKOEq9jbI8O1c5YUwowtVlPBNSlzRMuEhceJ1bYDWyUQk3zpVLaXy
+Z3mZtMq5NkAjidlol1ZtwMxvwDy478DjxNQZ7eR/coQmq2jj3tBeKH9AXOZlDQw
UHfmEmBwXHNK82Tp/2eW/Sk8psLngEsvAVPLexeS5QArs+wGPZp1cpV1iSc3AnVB
VOxaB4uzzTXUjP2H8Z68a34B8tMdej0MLHC1KUcWqgyi/Mdq6l8HeolBMUbcFzqA
vbVm8+6DhZPvc4F00bzlDvW23b2pI4RraI8fnEXHty6rfkJuHNVR+N8ZdaYZBODd
/n0a0fTQ1N361KFGr5EF7LX4qKJz2cP2m7qxSPmtZAgzGavUR1JDvCXzyjbPecWR
y0cuCmp8BC+Pd4s3y3b6tqNuharJfZSZ6B0eN99926J5ne7G1BmyPvPj7wb5KuW1
yKGn32DL/Bn+a4oReWngHMLDo/4xmxeJrpmtovwmJOXo5o+UeEU3ywr+sUBJc3W8
oUOXNfQwjdNXMkgVspf8w7bGecucFdmI0sDiYGNk5uvmwUjukfVLT9JPMN8hOns7
onw+9H+FYFUbEeWOu7QpqGRTZYoKJrXSrzII3YFmxE9u3UHLOqqDUIsHjHccmnqx
zRDSfkBkA6ItIqx55+cE0f0sdofXtvzvCRWBa5GFaBtNJhF940Lx9xfbdwOEZzBD
wYZvFv3c1VePTT0wvWybvo0qJTfauB1yRGM1l7ocB2wiHgZBTxPVDjb4qfVT8FNP
f17Dz/BjRDUIKoMu7gTifpnB+iw449cW2y538U+OmOqJE5myq+U0IkY9yydgDB6u
uGrfkAYp6NDvPF71PgiAhcrzggGuDq2jizoeH1Oq9yvt4pn3Q8d8EvuCs32464l5
O+2w+T2AeiPl74+xzkhGa1EcPJavpjogio0E5VAEavh6Yea/riHOHeMiQdQlM+tN
C6YOrVDEUicDGZGVoRROZ2gDbjh6xEZexqKc9Dmt9JbJfYobBG702VC7EpxiHGeJ
mJZ/cDXFDhJ1lBnkF8qhmTQtziEoEyB3D8yiUvW8xRaZGlOQnZWikyKGtJRIrGZv
OcD6BKQSzYoo36vNPK4U7QAVLRyNDHyeYTo8LzNsx0aDbu1rUC+83DyJwUIxOCmd
6WPCj80p/mnnjcF42wwgOVtXduekQBXZ5KpwvmXjb+yoyPCgJbiVwwUtmgZcUN8B
zQ8oFwPXTszUYgNjg5RFgj/MBYTraL6VYDAepn4YowdaAlv3M8ICRKQ3GbQEV6ZC
miDKAMx3K3VJpsY4aV52au5x43do6e3xyTSR7E2bfsUblzj2b+mZXrmxst+XDU6u
x1a9TrlunTcJJZJWKrMTEL4LRWPwR0tsb25tOuUr6DP/Hr52MLaLg1yIGR81cR+W
-----END RSA PRIVATE KEY-----' > id_rsa<code>

After doing so, we can use ssh2john to generate a hash and then use john to crack that hash.
![](ssh2john-hash.png)
![](rsa-hash-crack.png)

Now that I have the RSA passkey, I can login using SSH and the RSA key. Make sure to provide the RSA key using the `-i` flag.
![](ssh-login.png)

We can then find the `user.txt` flag in the home directory.
![](user-flag.png)

## Privilege Escalation

Now that we have the user flag, we can search for privilege escalation methods. It looks like we can run `sudo -l`, so I check what commands I can run with sudo. Turns out that I can run `cat` with sudo.
![](gtfobins-cat.png)
After checking gtfobins, I find a method to read files using cat. We'll use this method to read `/etc/shadow` and find the root password.
![](privilege-escalation.png)

We'll echo the root hash into another file and use john to crack it.
![](root-hash-crack.png)
We got our root password. Now we can switch users to root and provide the password. Presto! We're now root. I find the root flag in `/root/root.txt` and read its contents.

Now I've successfully rooted Brute It!