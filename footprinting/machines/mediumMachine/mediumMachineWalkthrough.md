# Enumerate the server carefully and find the username "HTB" and its password. Then, submit this user's password as the answer.

This second server is a server that everyone on the internal network has access to. In our discussion with our client, we pointed out that these servers are often one of the main targets for attackers and that this server should be added to the scope.

Our customer agreed to this and added this server to our scope. Here, too, the goal remains the same. We need to find out as much information as possible about this server and find ways to use it against the server itself. For the proof and protection of customer data, a user named HTB has been created. Accordingly, we need to obtain the credentials of this user as proof.

---

Let's ping the machine to be sure is up and running

`ping -c 1 10.129.202.41`

Output: 

````
PING 10.129.202.41 (10.129.202.41) 56(84) bytes of data.
64 bytes from 10.129.202.41: icmp_seq=1 ttl=127 time=70.2 ms

--- 10.129.202.41 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 70.179/70.179/70.179/0.000 ms
````

From the ttl we can already know is a Windwos machine. 

### Let's start with Nmap

`sudo nmap -F -T4 -sV -vv 10.129.202.41 -oX servicesAndVersionScan`

-F to scan the most used ports. T4 to accelerate the process (we don't care to be silent here), -sV for services and versions. -vv for double verbose. -oX to have an output on .xml file and then conver it on .html

Output: 

````
111/tcp  open  rpcbind       syn-ack ttl 127 2-4 (RPC #100000)
135/tcp  open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds? syn-ack ttl 127
2049/tcp open  nlockmgr      syn-ack ttl 127 1-4 (RPC #100021)
3389/tcp open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
````

Let's convert the .xml output from nmap on a .html redeable file. 

`xsltproc servicesAndVersionScan -o ServicesAndVersionsScan.html`

Now we can open the file on chromium or any other internet explorer

`chromium ServicesAndVersionsScan.html`

![alt text](image.png)

Let's try a simple vulnerability scan with nmap

`nmap --script vuln -p 111,135,139,445,2049,3389 -T4 10.129.202.41`

Output: 

````
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-20 14:22 CST
Nmap scan report for 10.129.202.41
Host is up (0.070s latency).

PORT     STATE SERVICE
111/tcp  open  rpcbind
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nfs
3389/tcp open  ms-wbt-server

Host script results:
|_smb-vuln-ms10-061: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
|_samba-vuln-cve-2012-1182: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
|_smb-vuln-ms10-054: false

Nmap done: 1 IP address (1 host up) scanned in 35.27 seconds
````

Nothing really useful. 

search exploit rpc

checked: 6, 

CVE 2017-8461?

10.129.202.41