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

We can use this private RSA key in conjunction with ssh2john to generate a hash that we can crack later. First, let's echo the private key into a new id_rsa file.
![](rsa_private.png)

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