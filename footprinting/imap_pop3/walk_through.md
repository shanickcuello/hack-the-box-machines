# Figure out the exact organization name from the IMAP/POP3 service and submit it as the answer.

> sudo nmap [TARGET_IP] -sC -sV -n -T5 -p- -oG allports_scan_common_scripts -v

The answer will be in the output. On the port 110
Search for this:

110/tcp   open  pop3     Dovecot pop3d
| ssl-cert: Subject: commonName=dev.inlanefreight.htb/organizationName=InlaneFreight Ltd/stateOrProvinceName=London/countryName=UK
| Issuer: commonName=`[SECOND ANSWER]`/organizationName=`[HERE IS THE ANSWER]`/stateOrProvinceName=London/countryName=UK
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-11-08T23:10:05
| Not valid after:  2295-08-23T23:10:05
| MD5:   ef51:134e:59ff:6733:43f6:b8b2:1ba7:c081
|_SHA-1: c841:8247:2db5:6a25:33eb:8e2f:a6ca:4680:cede:8f68
|_ssl-date: TLS randomness does not represent time

# What is the FQDN that the IMAP and POP3 servers are assigned to?
The answer we can get it from the same output that before:
| Issuer: commonName=dev.`[HERE IS THE SECOND ANSWER]`/organizationName=

# Enumerate the IMAP service and submit the flag as the answer. (Format: HTB{...})
From the output of the file [all_ports](https://github.com/shanickcuello/hack-the-box-machines/blob/main/footprinting/imap_pop3/allports_scan_common_scripts) we know the port 143 is handling IMAP. As you can see here:
143/open/tcp//imap//Dovecot imapd/

So we run:

`telnet [TARGET_IP] 143`

And we get this output:
Trying [TARGET_IP]...
Connected to [TARGET_IP]
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LITERAL+ STARTTLS LOGINDISABLED]
HTB{[FLAG]}
Submit the Flag.

# What is the customized version of the POP3 server?
We know from our allports file that pop3 is running in ports 995
So we run:

`openssl s_client -connect [TARGET_IP]:995`

We will have this output at the end:


    00a0 - ed e2 80 7d 6b 87 eb 9b-19 a0 01 d4 d2 9d ca 8b   ...}k...........
    00b0 - 7a 6e bd 51 cb 02 2c 38-be 99 b6 ca ec 82 5c 83   zn.Q..,8......\.

    Start Time: 1726355673
    Timeout   : 7200 (sec)
    Verify return code: 18 (self-signed certificate)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
+OK [HERE IS THE ANSWER]

# What is the admin email address?
To know from who is that email, we can run: 
> A2 FETCH 1 BODY[HEADER]
This is the output:

    * 1 FETCH (BODY[HEADER] {133}
    Subject: Flag
    To: Robin <robin@inlanefreight.htb>
    From: CTO <HERE_IS_THE_ANSWER>
    Date: Wed, 03 Nov 2021 16:13:27 +0200
    
    )
    A2 OK Fetch completed (0.001 + 0.000 secs).
You will find the admin email from there. 


#  Try to access the emails on the IMAP server and submit the flag as the answer. (Format: HTB{...})

Because we know from the file allports that port 993 has imap running we will connect with: 
> openssl s_client -connect [TARGET_IP]:993

Then we will list the folders:

> A1 LIST "" *

    * LIST (\Noselect \HasChildren) "." DEV
    * LIST (\Noselect \HasChildren) "." DEV.DEPARTMENT
    * LIST (\HasNoChildren) "." DEV.DEPARTMENT.INT
    * LIST (\HasNoChildren) "." INBOX

As we can see, there is a structure of "folders". Like this: DEV/DEPARTMENT/INT
and /INBOX

Let's see what /././INT has inside:
> A1 SELECT DEV.DEPARTMENT.INT

    * FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
    * OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
    * 1 EXISTS
    * 0 RECENT
    * OK [UIDVALIDITY 1636414279] UIDs valid
    * OK [UIDNEXT 2] Predicted next UID
    A1 OK [READ-WRITE] Select completed (0.006 + 0.000 + 0.005 secs).

As we can see it says "1 exists" that means there is some email inside there.

Last, we will read the email:
A1 SELECT DEV.DEPARTMENT.INT

    * 1 FETCH (BODY[TEXT] {34}
    HTB{FLAG}
    )
    A4 OK Fetch completed (0.004 + 0.000 + 0.003 secs).

And that's the flag.



