# Hack The Box

This repository is a curated collection of **network footprinting resources**, **scan outputs**, and **walkthroughs** for Hack The Box machines and common service types.

## Structure

The repository is organized by service or protocol, each folder containing specific tools, outputs or notes:

```
footprinting/
├── imap_pop3/         # Scans or enumeration related to IMAP and POP3 services
├── ipmi/              # IPMI-specific scanning techniques and scripts
├── machines/          # Walkthroughs and reconnaissance notes for specific HTB machines
├── mssql/             # Scans and enumeration techniques for Microsoft SQL Server
├── my_sql_databases/  # MySQL-related enumeration data and tools
├── oracle_tns/        # Oracle TNS listener scan outputs and notes
├── snmp/              # SNMP enumeration outputs and walk-throughs
├── Footprinting-wordlist.zip  # Custom or compiled wordlist for directory or service brute-forcing
```

## Goal

This project serves as a **knowledge base and reference toolkit** for red teamers and penetration testers practicing on platforms like **Hack The Box**.

It includes:
- Real-world scan outputs (`nmap`, `onesixtyone`, `enum4linux`, etc.)
- Protocol-specific enumeration results
- Structured directories for efficient learning and reuse
- Wordlists and automation-ready outputs
