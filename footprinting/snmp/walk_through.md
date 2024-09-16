# Enumerate the SNMP service and obtain the email address of the admin. Submit it as the answer.

### nmap

We send 2 ping to verify host is up

> ping [TARGET_IP] -c 2

We start with a Nmap scan.

-sV to discover service versions

-v to have outputs during the scan

-T5 to do it fast. (This creates a lot of noise)

-oA to have all the exports formats

-sU to scan UDP since snmp runs normally there

-p- to scan all ports

--script=snmp-info to use snmp script fron nmap that will run some specific scan for this service

> sudo nmap -sU -p- --script=snmp-info [TARGET_IP] -v T5 -sV -oA snmp_script_scan

nmap was taking too much time. Decided to switch strategy

![image](https://github.com/user-attachments/assets/04d81d07-a7dc-43e8-b12b-75ec66148ec1)

### snmpwalk

> snmpwalk -v2c -c public 10.129.238.27 > snmpwalk_scan.txt

In the [export](https://github.com/shanickcuello/hack-the-box-machines/blob/main/footprinting/snmp/snmpExports/snmpwalk_scan.txt) we can see the email that the exercises requires. 

> cat snmpwalk_scan.txt

    iso.3.6.1.2.1.1.1.0 = STRING: "Linux NIX02 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64"
    iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
    iso.3.6.1.2.1.1.3.0 = Timeticks: (75057) 0:12:30.57
    iso.3.6.1.2.1.1.4.0 = STRING: "devadmin [HERE_IS_THE_ANSWER]"


# What is the customized version of the SNMP server?
We also have this answer in the same [export](https://github.com/shanickcuello/hack-the-box-machines/blob/main/footprinting/snmp/snmpExports/snmpwalk_scan.txt).

    iso.3.6.1.2.1.1.5.0 = STRING: "NIX02"
    iso.3.6.1.2.1.1.6.0 = STRING: [HERE_IS_THE_ANSWER]
    iso.3.6.1.2.1.1.7.0 = INTEGER: 72
    iso.3.6.1.2.1.1.8.0 = Timeticks: (37) 0:00:00.37


# Enumerate the custom script that is running on the system and submit its output as the answer.

If we use grep to search a `.sh` file, we will encounter this:

> grep -i '.sh' snmpwalk_scan.txt

    iso.3.6.1.2.1.25.1.7.1.2.1.2.4.70.76.65.71 = STRING: "/usr/share/flag.sh"
    iso.3.6.1.2.1.25.2.3.1.3.8 = STRING: "Shared memory"
    iso.3.6.1.2.1.25.2.3.1.3.38 = STRING: "/dev/shm"
    iso.3.6.1.2.1.25.3.8.1.2.8 = STRING: "/dev/shm"
    iso.3.6.1.2.1.25.4.2.1.2.241 = STRING: "kdmflush"
    iso.3.6.1.2.1.25.4.2.1.2.243 = STRING: "kdmflush"
    iso.3.6.1.2.1.25.4.2.1.2.1000 = STRING: "sshd"
    iso.3.6.1.2.1.25.4.2.1.4.1000 = STRING: "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
    iso.3.6.1.2.1.25.6.3.1.2.14 = STRING: "bash_5.0-6ubuntu1.1_amd64"
    iso.3.6.1.2.1.25.6.3.1.2.15 = STRING: "bash-completion_1:2.10-1ubuntu1_all"
    iso.3.6.1.2.1.25.6.3.1.2.47 = STRING: "dash_0.5.10.2-6_amd64"
    iso.3.6.1.2.1.25.6.3.1.2.378 = STRING: "libssh-4_0.9.3-2ubuntu2.2_amd64"
    iso.3.6.1.2.1.25.6.3.1.2.452 = STRING: "lshw_02.18.85-0.3ubuntu2.20.04.1_amd64"
    iso.3.6.1.2.1.25.6.3.1.2.491 = STRING: "openssh-client_1:8.2p1-4ubuntu0.3_amd64"
    iso.3.6.1.2.1.25.6.3.1.2.492 = STRING: "openssh-server_1:8.2p1-4ubuntu0.3_amd64"
    iso.3.6.1.2.1.25.6.3.1.2.493 = STRING: "openssh-sftp-server_1:8.2p1-4ubuntu0.3_amd64"
    iso.3.6.1.2.1.25.6.3.1.2.603 = STRING: "shared-mime-info_1.15-1_amd64"
    iso.3.6.1.2.1.25.6.3.1.2.612 = STRING: "squashfs-tools_1:4.4-1ubuntu0.3_amd64"
    iso.3.6.1.2.1.25.6.3.1.2.613 = STRING: "ssh-import-id_5.10-0ubuntu1_all"

As you can see we found a /usr/share/flag.sh

Let's see the previous and after 10 lines of that output: 

> grep -A 10 -B 10 '/usr/share/flag.sh' snmpwalk_scan.txt

    iso.3.6.1.2.1.11.32.0 = Counter32: 0
    iso.3.6.1.2.1.25.1.1.0 = Timeticks: (81566) 0:13:35.66
    iso.3.6.1.2.1.25.1.2.0 = Hex-STRING: 07 E8 09 10 13 04 17 00 2B 00 00
    iso.3.6.1.2.1.25.1.3.0 = INTEGER: 393216
    iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/vmlinuz-5.4.0-90-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro ipv6.disable=1 maybe-ubiquity
    "
    iso.3.6.1.2.1.25.1.5.0 = Gauge32: 0
    iso.3.6.1.2.1.25.1.6.0 = Gauge32: 144
    iso.3.6.1.2.1.25.1.7.0 = INTEGER: 0
    iso.3.6.1.2.1.25.1.7.1.1.0 = INTEGER: 1
    iso.3.6.1.2.1.25.1.7.1.2.1.2.4.70.76.65.71 = STRING: "/usr/share/flag.sh"
    iso.3.6.1.2.1.25.1.7.1.2.1.3.4.70.76.65.71 = ""
    iso.3.6.1.2.1.25.1.7.1.2.1.4.4.70.76.65.71 = ""
    iso.3.6.1.2.1.25.1.7.1.2.1.5.4.70.76.65.71 = INTEGER: 5
    iso.3.6.1.2.1.25.1.7.1.2.1.6.4.70.76.65.71 = INTEGER: 1
    iso.3.6.1.2.1.25.1.7.1.2.1.7.4.70.76.65.71 = INTEGER: 1
    iso.3.6.1.2.1.25.1.7.1.2.1.20.4.70.76.65.71 = INTEGER: 4
    iso.3.6.1.2.1.25.1.7.1.2.1.21.4.70.76.65.71 = INTEGER: 1
    iso.3.6.1.2.1.25.1.7.1.3.1.1.4.70.76.65.71 = STRING: "HTB{FLAG}"
    iso.3.6.1.2.1.25.1.7.1.3.1.2.4.70.76.65.71 = STRING: "HTB{FLAG}"
    iso.3.6.1.2.1.25.1.7.1.3.1.3.4.70.76.65.71 = INTEGER: 1

There you have the flag. New spell mastered! 🧙
