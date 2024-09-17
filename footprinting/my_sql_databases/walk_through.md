# Enumerate the MySQL server and determine the version in use. (Format: MySQL X.X.XX)
> sudo nmap 10.129.35.132 -sV -sC -F --script 'mysql*' -oA nmap_scan_100_ports_mysql -v

    -sV for service version
    -sC for common script scan
    -F for most important ports
    --script mysql* for sql scan script
    -oA for export everything in files
    -v to see output and progress of nmap

(Note: mysql* has '' because im using [fish shell](https://fishshell.com/docs/current/language.html#expand-wildcard) and the wildcard is interpreted differntly.)

Let's make those outputs redeables. 

> xmltohtml nmap_scan_100_ports_mysql.xml > nmap_scan_100_ports_mysql.html
Note: `xmltohtml` is an alias for [xsltproc](https://github.com/ilyar/xsltproc/blob/main/README.md). You can do the same with the command: xsltproc -o output.html transform.xsl file.xml

If we open the report: 

> chromium nmap_scan_100_ports_mysql.html

We can see the version right there

![image](https://github.com/user-attachments/assets/4dee6a19-9cfa-4252-9de7-25636d7d40ce)

--- 

# During our penetration test, we found weak credentials "robin:robin". We should try these against the MySQL server. What is the email address of the customer "Otto Lang"h?

Let's connect with the given credentials:

> mysql -u robin -probin -h 10.129.35.132

Let's reed those tables:

> MySQL [(none)]> use sys;
> MySQL [sys]> show databases;

we get this output:
    
    +--------------------+
    | Database           |
    +--------------------+
    | customers          |
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    
Let's select customers:
> MySQL [sys]> USE customers;

    Reading table information for completion of table and column names
    You can turn off this feature to get a quicker startup with -A
    
    Database changed

Let's see the tables:

> MySQL [customers]> show tables

    +---------------------+
    | Tables_in_customers |
    +---------------------+
    | myTable             |
    +---------------------+
    1 row in set (0.465 sec)

Let's see the content: 

> MySQL [customers]> SELECT * FROM myTable;

Ok. Too much content. But know we can find the specific value that was asked. 

> MySQL [customers]> SELECT * FROM myTable WHERE name = 'Otto Lang';

Done! This is what we get:

    +----+-----------+---------------------+---------+-----------+---------+-----------------+------------------+------+
    | id | name      | email               | country | postalZip | city    | address         | pan              | cvv  |
    +----+-----------+---------------------+---------+-----------+---------+-----------------+------------------+------+
    | 88 | Otto Lang | [THIS IS THE ANSWER] | France  | 76733-267 | Belfast | 4708 Auctor Rd. | 5322224628183391 | 595  |
    +----+-----------+---------------------+---------+-----------+---------+-----------------+------------------+------+
    1 row in set (0.125 sec)

There you have the last answer. New spell mastered! 🧙
