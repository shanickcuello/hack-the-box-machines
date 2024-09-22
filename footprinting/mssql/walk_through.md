# Enumerate the target using the concepts taught in this section. List the hostname of MSSQL server.

Let's use metasploit for this one

> msfconsole

> use mssql_ping

Let's run `info` to see what is necessary to set :
    
    [msf](Jobs:0 Agents:0) auxiliary(scanner/mssql/mssql_ping) >> info
    
           Name: MSSQL Ping Utility
         Module: auxiliary/scanner/mssql/mssql_ping
        License: Metasploit Framework License (BSD)
           Rank: Normal
    
    Provided by:
      MC <mc@metasploit.com>
    
    Check supported:
      No
    
    Basic options:
      Name                 Current Setting  Required  Description
      ----                 ---------------  --------  -----------
      PASSWORD                              no        The password for the specified username
      RHOSTS                                yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.h
                                                      tml
      TDSENCRYPTION        false            yes       Use TLS/SSL for TDS data "Force Encryption"
      THREADS              1                yes       The number of concurrent threads (max one per host)
      USERNAME             sa               no        The username to authenticate as
      USE_WINDOWS_AUTHENT  false            yes       Use windows authentication (requires DOMAIN option set)
    
    Description:
      This module simply queries the MSSQL instance for information.
    
    
    View the full module info with the info -d command.

We set the host:
> set RHOST [TARGET_IP]
> exploit

After exploit, this is the output we get
    
    [*] 10.129.230.249:       - SQL Server information for 10.129.230.249:
    [+] 10.129.230.249:       -    ServerName      = [THIS_IS_THE_ANSWER]
    [+] 10.129.230.249:       -    InstanceName    = MSSQLSERVER
    [+] 10.129.230.249:       -    IsClustered     = No
    [+] 10.129.230.249:       -    Version         = 15.0.2000.5
    [+] 10.129.230.249:       -    tcp             = 1433
    [+] 10.129.230.249:       -    np              = \\ILF-SQL-01\pipe\sql\query
    [*] 10.129.230.249:       - Scanned 1 of 1 hosts (100% complete)
    [*] Auxiliary module execution completed

# Connect to the MSSQL instance running on the target using the account (backdoor:Password1), then list the non-default database present on the server.

Let's use impacket to connect us to the mssql service. 

> impacket-mssqlclient backdoor:Password1@[TARGET_IP] -windows-auth

You will find an output like this:

    Impacket v0.11.0 - Copyright 2023 Fortra    
    [*] Encryption required, switching to TLS
    [*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
    [*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
    [*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
    [*] INFO(ILF-SQL-01): Line 1: Changed database context to 'master'.
    [*] INFO(ILF-SQL-01): Line 1: Changed language setting to us_english.
    [*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
    [!] Press help for extra shell commands

Now let's get those DBs: 

> SQL (ILF-SQL-01\backdoor  dbo@master)> enum_db

This is the output:

    master                      0   

    tempdb                      0   
    
    model                       0   
    
    msdb                        1   
    
    [THIS IS THE ANSWER]        0   

New spell mastered! 🧙
