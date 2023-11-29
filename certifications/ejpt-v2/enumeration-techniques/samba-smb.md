# Samba (SMB)

* smb -> server message block
* net use \* /delete -> deletes all the folders and files in SMB
* net use z: \\\\\<IP>\dir$ \<password> \<username> -> will create a samba share

### nmap scans

* session enumeration&#x20;

```bash
nmap $IP -p445 --script smb-enum-sessions \
--script-args smbusername=$username,smbpassword=$password
```

* share enumerations

```bash
nmap $IP -p445 --script smb-enum-shares \
--script-args smbusername=$username,smbpassword=$password
```

* enumerate users -> to find how many users exists

```bash
nmap $IP -p445 --script smb-enum-users \
--script-args smbusername=$username,smbpassword=$password
```

* enumerate server staticstics -> prints out how many logged in, failed attempt count, etc.

```bash
nmap $IP -p445 --script smb-enum-stats \
--script-args smbusername=$username,smbpassword=$password
```

* enumerate domain (scan) -> find out how many domains are there

```bash
nmap $IP -p445 --script smb-enum-domains \
--script-args smbusername=$username,smbpassword=$password
```

* enumerate groups -> shows all users belongs to what groups

```bash
nmap $IP -p445 --script smb-enum-groups \
--script-args smbusername=$username,smbpassword=$password
```

* enumerate services -> to see what services are running

```bash
nmap $IP -p445 --script smb-enum-services \
--script-args smbusername=$username,smbpassword=$password
```

```bash
nmap $IP -p445 --script smb-enum-services,smb-ls \
--script-args smbusername=$username,smbpassword=$password

# above command just with `smb-ls` which will list all the files
# smb-ls -> linux machine
# smb-dir -> windows machine
```

* smb protocols or Dialects

```bash
nmap $IP -p445 --script smb-protocols
```

* smb security level information

```bash
nmap $IP -p445 --script smb-security-mode
```



### smb map

* to list , see what permission we have

```bash
smbmap -u guest -p "" -d . -H $IP
```

```bash
smbmap -u $username -p $password -d . -H $IP
```

* To run commands on the target machine (-x flag)

```bash
smbmap -u $username -p $password -H $IP -x $command
```

* To list shares

```bash
smbmap -u $username -p $password -H $IP -L
```

* to see contents of the shares

```bash
smbmap -u $username -p $password -H $IP -r <dir> #e.g. C$ 
```

* to upload to the smb share

```bash
smbmap -u $username -p $password --upload '/local/machine/file' '$C/remote/machine'
# if windows : C$\file\path
```

* to download the smb share

```bash
smbpmap -u $username -p $password --download '$C\path\file'
```



### linux enum

* msfconsole module to scan samba version

```bash
msfconsole
use auxillary/scanner/smb/smb_version 

#run : 'show options' & fill up needed fields.
```

### nmblookup





















