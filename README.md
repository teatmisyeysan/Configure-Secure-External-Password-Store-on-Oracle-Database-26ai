# Configure-Secure-External-Password-Store-on-Oracle-Database-26ai
## Overviews
The Oracle Wallet can be used to store the user's credentials, so instead of exposing passwords in clear text format in a shell script. Multiple credentials for multiple database can be stored in a single wallet file.

This procedure stores a database user's credentials inside the Wallet. This features uses the auto login feature, so it is not required to provide the Wallet password to access to the user's credentials, the OS file permissions regulate access to the Wallet.

## 1). Create Oracle Wallet Directory
```sql
[oracle@ms-vm-01 ~]$ ps -ef |grep pmon
grid        3098       1  0 21:55 ?        00:00:00 asm_pmon_+ASM
oracle      3427       1  0 21:56 ?        00:00:00 ora_pmon_cdb26ai
oracle     13462    2306  0 22:19 pts/0    00:00:00 grep --color=auto pmon
[oracle@ms-vm-01 ~]$
[oracle@ms-vm-01 ~]$
[oracle@ms-vm-01 ~]$ ls -ltr /u01/app/oracle/admin/$ORACLE_SID
total 0
drwxr-x---. 2 oracle asmdba    44 Aug 27 22:51 xdb_wallet
drwxr-x---. 2 oracle oinstall  36 Aug 27 22:56 pfile
drwxr-x---. 5 oracle oinstall 140 Sep  3 12:00 dpdump
[oracle@ms-vm-01 ~]$
[oracle@ms-vm-01 ~]$ mkdir -p /u01/app/oracle/admin/$ORACLE_SID/wallet
[oracle@ms-vm-01 ~]$
[oracle@ms-vm-01 ~]$ ls -ltr /u01/app/oracle/admin/$ORACLE_SID
total 0
drwxr-x---. 2 oracle asmdba    44 Aug 27 22:51 xdb_wallet
drwxr-x---. 2 oracle oinstall  36 Aug 27 22:56 pfile
drwxr-x---. 5 oracle oinstall 140 Sep  3 12:00 dpdump
drwxr-xr-x. 2 oracle oinstall   6 Sep 14 22:20 wallet
[oracle@ms-vm-01 ~]$
[oracle@ms-vm-01 ~]$
```

## 2). Create a new wallet following syntex if not exist
```sql
[oracle@ms-vm-01 wallet]$
[oracle@ms-vm-01 wallet]$ mkstore -wrl /u01/app/oracle/admin/$ORACLE_SID/wallet/ -create
Oracle Secret Store Tool Release 23.0.0.0.0 - Production
Version 23.0.0.0.0
Copyright (c) 2004, 2026, Oracle and/or its affiliates. All rights reserved.

Enter password:
Enter password again:
[oracle@ms-vm-01 wallet]$
[oracle@ms-vm-01 wallet]$ ls -ltr
total 8
-rw-------. 1 oracle oinstall   0 Sep 14 22:32 ewallet.p12.lck
-rw-------. 1 oracle oinstall 225 Sep 14 22:32 ewallet.p12
-rw-------. 1 oracle oinstall   0 Sep 14 22:32 cwallet.sso.lck
-rw-------. 1 oracle oinstall 270 Sep 14 22:32 cwallet.sso
[oracle@ms-vm-01 wallet]$
[oracle@ms-vm-01 wallet]$
```

## 3). Add source tns entries in client tnsnames.ora file
```sql
vi /u01/app/oracle/product/26.0.0/dbhome_1/network/admin/tnsnames.ora

Add:
# External Password Store
DBA_MISY =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 10.xxx.xxx.111)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = AI26PDB1)
    )
  )


TNSPING:
[oracle@ms-vm-01 admin]$ tnsping DBA_MISY

TNS Ping Utility for Linux: Version 23.26.1.0.0 - Production on 14-SEP-2026 22:38:17

Copyright (c) 1997, 2026, Oracle.  All rights reserved.

Used parameter files:


Used TNSNAMES adapter to resolve the alias
Attempting to contact (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = 10.xxx.xxx.111)(PORT = 1521)) (CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = AI26PDB1)))
OK (0 msec)
[oracle@ms-vm-01 admin]$
```
On sqlnet.ora Add:
```sql
[oracle@ms-vm-01 admin]$
[oracle@ms-vm-01 admin]$ cat sqlnet.ora
NAMES.DIRECTORY_PATH= (TNSNAMES)
SQLNET.WALLET_OVERRIDE = TRUE
SSL_CLIENT_AUTHENTICATION = FALSE
SSL_VERSION = 0
WALLET_LOCATION=(SOURCE=(METHOD=FILE)(METHOD_DATA=(DIRECTORY=/u01/app/oracle/admin/$ORACLE_SID/wallet)))
[oracle@ms-vm-01 admin]$
```

## 4). Create User on oracle database
```sql
SQL> create user dba_misy identified by Welcome1;
[oracle@ms-vm-01 admin]$ sqlplus dba_misy/Welcome1@AI26PDB1

SQL*Plus: Release 23.26.1.0.0 - Production on Mon Sep 14 22:39:20 2026
Version 23.26.1.0.0

Copyright (c) 1982, 2025, Oracle.  All rights reserved.

Last Successful login time: Mon Sep 14 2026 21:59:12 +07:00

Connected to:
Oracle AI Database 26ai Enterprise Edition Release 23.26.1.0.0 - Production
Version 23.26.1.0.0

SQL> show user;
USER is "DBA_MISY"
SQL>
```
## 5). Create database connection credentials in the wallet
```sql
[oracle@ms-vm-01 admin]$ mkstore -wrl /u01/app/oracle/admin/$ORACLE_SID/wallet/ -createCredential DBA_MISY dba_misy Welcome1
Oracle Secret Store Tool Release 23.0.0.0.0 - Production
Version 23.0.0.0.0
Copyright (c) 2004, 2026, Oracle and/or its affiliates. All rights reserved.

Enter wallet password:
[oracle@ms-vm-01 admin]$
```
## 6). List Credentials
```sql
[oracle@ms-vm-01 admin]$
[oracle@ms-vm-01 admin]$ mkstore -wrl /u01/app/oracle/admin/$ORACLE_SID/wallet/ -listCredential
Oracle Secret Store Tool Release 23.0.0.0.0 - Production
Version 23.0.0.0.0
Copyright (c) 2004, 2026, Oracle and/or its affiliates. All rights reserved.

Enter wallet password:
List credential (index: connect_string username)
1: DBA_MISY dba_misy
[oracle@ms-vm-01 admin]$
```

## 7). Connect DBA_MISY using without supplying password from client machine using wallet
```sql
[oracle@ms-vm-01 admin]$
[oracle@ms-vm-01 admin]$ sqlplus /@DBA_MISY

SQL*Plus: Release 23.26.1.0.0 - Production on Mon Sep 14 22:59:56 2026
Version 23.26.1.0.0

Copyright (c) 1982, 2025, Oracle.  All rights reserved.

Last Successful login time: Mon Sep 14 2026 22:39:20 +07:00

Connected to:
Oracle AI Database 26ai Enterprise Edition Release 23.26.1.0.0 - Production
Version 23.26.1.0.0

SQL> show user;
USER is "DBA_MISY"
SQL>
```
