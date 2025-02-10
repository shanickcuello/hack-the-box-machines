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

-p- to scan all ports. -T4 to accelerate the process (we don't care to be silent here), -sV for services and versions. -vvv for verbose. -oX to have an output on .xml file and then conver it on .html

Output: 

````
PORT      STATE SERVICE       REASON          VERSION
111/tcp   open  rpcbind       syn-ack ttl 127 2-4 (RPC #100000)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 127
2049/tcp  open  nlockmgr      syn-ack ttl 127 1-4 (RPC #100021)
3389/tcp  open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
47001/tcp open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49679/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49680/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49681/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
````

Let's convert the .xml output from nmap on a .html redeable file. 

`xsltproc servicesAndVersionScan -o ServicesAndVersionsScan.html`
![image](https://github.com/user-attachments/assets/4e0ddc13-01c0-412d-90dc-6c77e138a079)

`2049/tcp` is the standard NFS (Network File System) port.
nlockmgr is a service used to manage file locks on NFS and other systems using RPC.
Although the scan did not explicitly detect "nfs", the presence of rpcbind (port 111/tcp) + nlockmgr (2049/tcp) suggests that NFS might be running in the background.
Let's see if it's running: 

`showmount -e 10.129.198.99` 

Output:
````
Export list for 10.129.198.99:
/TechSupport (everyone)
````

There you are... We have a NFS service running with the directory `/TechSupport` that can be access for everyone.

Change to root user
`sudo su -i`

Let's create a folder for the mounted nfs
`mkdir target-NFS`

Mount it
`sudo mount -t nfs 10.129.198.99:/ ./target-NFS/ -o nolock`

Let's dig inside
`cd target-NFS`
`cd TechSupport`

Output: 
![image](https://github.com/user-attachments/assets/56142cee-304a-4cb2-8252-d9719098d863)

We can see a lot of .txt files. Let's see what's inside with cat command and pip it to less so we can navigate
`cat *.txt | less` 

Output:
![image](https://github.com/user-attachments/assets/7998d7e0-55b3-4d7d-87c5-da8a2a5a32de)

We have a user on line 5 and password on line 6. That should help us get into some service/database. 

Or cat the specified file with data inside:
`cat ticket4238791283782.txt` 
We know this is the file that has data because of the size when we run `ls -la`





