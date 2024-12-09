
# BorazuwarahCTF
![image](https://github.com/user-attachments/assets/30562465-cc76-411d-9c42-d8cf6859c980)

## Recognition and Enumeration.

Check that the machine is running.

````bash
❯ ping -c 1 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.036 ms

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.036/0.036/0.036/0.036/0.000 ms

```

Once we get a ping response, we can start enumerating the machine. The ttl=64 may indicate that it is a **Linux** machine.

Launch nmap to see what ports it finds.

````bash
nmap -sS -p- --open --min-rate 5000 -vvv -n -Pn -oG allPorts 172.17.0.2
```
```
- `-sS`: Perform a SYN scan.
- `-p-`: Scan all ports.
- `--open`: Displays only open ports.
- `--min-rate 5000`: Sets the minimum packet sending rate to 5000 per second.
- `-vvv`: Generates very detailed and verbose output.
- `-n`: Prevents reverse DNS resolution.
- Pn`: Omit host discovery.
- `-oG allPorts`: Saves the results to a file named `-allPorts`.
- 172.17.0.2: The IP address of the target host.
```
```
PORT STATE SERVICE REASON
22/tcp open ssh syn-ack ttl 64
80/tcp open http syn-ack ttl 64
```
Since we have port 22 and 80 open, let's check the version and run some basic scripts to see what information we get.

````bash
nmap -sCV -p22,80 -vvv -oN versi

Translated with DeepL.com (free version)
