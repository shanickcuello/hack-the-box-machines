### Figure out the exact organization name from the IMAP/POP3 service and submit it as the answer.

> sudo nmap 10.129.157.249 -sC -sV -n -T5 -p- -oG allports_scan_common_scripts -v

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

### What is the FQDN that the IMAP and POP3 servers are assigned to?
The answer we can get it from the same output that before:
| Issuer: commonName=dev.`[HERE IS THE SECOND ANSWER]`/organizationName=

