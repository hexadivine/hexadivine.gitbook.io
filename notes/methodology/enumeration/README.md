
# [Introduction]()

Enumeration is the process of gathering detailed information about a target system, often through scanning or probing. This step helps identify potential vulnerabilities, user accounts, and network configurations for further analysis or exploitation.

## [Scanning with nmap]()

I scan for all ports recursively first

```
┌──[hexadivine@linux]─[~]
└──╼ $ sudo nmap -sS -T5 -p- -v -r 192.168.0.102  | grep open
Discovered open port 22/tcp on 192.168.0.102
Discovered open port 80/tcp on 192.168.0.102
Discovered open port 111/tcp on 192.168.0.102
Discovered open port 139/tcp on 192.168.0.102
Discovered open port 443/tcp on 192.168.0.102
Discovered open port 32768/tcp on 192.168.0.102

22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
443/tcp   open  https
32768/tcp open  filenet-tms
```

- **`-sS`**: Performs a **TCP SYN scan**, sending SYN packets to check for open ports without completing the handshake.
- **`-T5`**: Uses the **most aggressive timing template** (fastest scan), likely increasing the chances of detection.
- **`-p-`**: Scans **all 65,535 ports** (from port 1 to 65535) on the target.
- **`-v`**: Enables **verbose output**, providing more detailed information during the scan.
- **`-r`**: Scans **ports in raw order** (from 1 to 65535), instead of randomly scanning them.

After knowing these ports are open we can scan vertically.

```
┌──[hexadivine@linux]─[~]
└──╼ $ nmap -A -T5 -p22,80,111,139,443,32768 192.168.0.102
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-04 19:58 IST
Nmap scan report for 192.168.0.102
Host is up (0.00034s latency).

PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 2.9p2 (protocol 1.99)
|_sshv1: Server supports SSHv1
| ssh-hostkey: 
|   1024 b8:74:6c:db:fd:8b:e6:66:e9:2a:2b:df:5e:6f:64:86 (RSA1)
|   1024 8f:8e:5b:81:ed:21:ab:c1:80:e1:57:a3:3c:85:c4:71 (DSA)
|_  1024 ed:4e:a9:4a:06:14:ff:15:14:ce:da:3a:80:db:e2:81 (RSA)
80/tcp    open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
|_http-title: Test Page for the Apache Web Server on Red Hat Linux
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
111/tcp   open  rpcbind     2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1          32768/tcp   status
|_  100024  1          32768/udp   status
139/tcp   open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp   open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_64_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-09-26T09:32:06
|_Not valid after:  2010-09-26T09:32:06
|_ssl-date: 2024-12-04T19:29:00+00:00; +4h59m58s from scanner time.
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_http-title: 400 Bad Request
32768/tcp open  status      1 (RPC #100024)
MAC Address: 08:00:27:45:BB:E6 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.4.X
OS CPE: cpe:/o:linux:linux_kernel:2.4
OS details: Linux 2.4.9 - 2.4.18 (likely embedded)
Network Distance: 1 hop

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: 4h59m57s
|_nbstat: NetBIOS name: KIOPTRIX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

TRACEROUTE
HOP RTT     ADDRESS
1   0.34 ms 192.168.0.102

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.50 seconds
```

- **`-A`**: Performs **aggressive scanning**, including OS detection (`-O`), version detection(`-sV`), script scanning (`-sC`), and traceroute (`--traceroute`).
- **`-p22,80...`**: Scans **ports 22 (SSH)** and **80 (HTTP)** only, focusing on these specific services.

## [Enumerating HTTP/HTTPS]()

### [Basic Enumeration]()

- Default webpage tells architecture, version (`Apache/1.3.20`) aka information disclosure.

![](assets/Pasted%20image%2020241204201020.png)

![](assets/Pasted%20image%2020241204200831.png)

### [Fuff]()

#### [Bruteforce directories]()

This is used to burteforce the directories/files/subdomains. Use FUZZ keyword as a placeholder for words from wordlist. (`-w`)

```
┌──[hexadivine@linux]─[~]
└──╼ $ ffuf -u http://192.168.0.102/FUZZ -w /usr/share/wordlists/dirb/big.txt 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.0.102/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.htaccess               [Status: 403, Size: 273, Words: 20, Lines: 11, Duration: 4ms]
.htpasswd               [Status: 403, Size: 273, Words: 20, Lines: 11, Duration: 4ms]
cgi-bin/                [Status: 403, Size: 272, Words: 20, Lines: 11, Duration: 11ms]
manual                  [Status: 301, Size: 294, Words: 19, Lines: 10, Duration: 10ms]
mrtg                    [Status: 301, Size: 292, Words: 19, Lines: 10, Duration: 10ms]
usage                   [Status: 301, Size: 293, Words: 19, Lines: 10, Duration: 9ms]
~operator               [Status: 403, Size: 273, Words: 20, Lines: 11, Duration: 6ms]
~root                   [Status: 403, Size: 269, Words: 20, Lines: 11, Duration: 6ms]
:: Progress: [20469/20469] :: Job [1/1] :: 4545 req/sec :: Duration: [0:00:06] :: Errors: 0 ::
```

#### [Bruteforce subdomains]()

```
┌──[hexadivine@linux]─[~]
└──╼ $ ffuf -c -u http://linkvortex.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H 'HOST: FUZZ.linkvortex.htb' -fc 301

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://linkvortex.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.linkvortex.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 301
________________________________________________

dev                     [Status: 200, Size: 2538, Words: 670, Lines: 116, Duration: 205ms]
:: Progress: [4989/4989] :: Job [1/1] :: 294 req/sec :: Duration: [0:00:19] :: Errors: 0 ::
```

- `-H "HOST: FUZZ.linkvortex.htb"`: This option sets a custom header for the fuzzing requests. The `HOST` header is being set to `FUZZ.linkvortex.htb`, where `FUZZ` will be replaced with each word from the specified wordlist during the fuzzing process.
- `-fc 301`: This option tells `ffuf` to filter the results based on the status code. In this case, it will only display results where the status code is not `301`, which indicates a permanent redirect.
## [Enumerating SMB]()

### [Version discovery]()

```
[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> options

Module options (auxiliary/scanner/smb/smb_version):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   THREADS  1                yes       The number of concurrent threads (max one per host)


View the full module info with the info, or info -d command.

[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> set rhosts 192.168.0.102
rhosts => 192.168.0.102
[msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_version) >> run

[*] 192.168.0.102:139     - SMB Detected (versions:) (preferred dialect:) (signatures:optional)
[*] 192.168.0.102:139     -   Host could not be identified: Unix (Samba 2.2.1a)
[*] 192.168.0.102:        - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### [SMBClient]()

```
┌──[hexadivine@linux]─[~]
└──╼ $ smbclient -L \\\\192.168.0.102
Server does not support EXTENDED_SECURITY  but 'client use spnego = yes' and 'client ntlmv2 auth = yes' is set
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	IPC$            IPC       IPC Service (Samba Server)
	ADMIN$          IPC       IPC Service (Samba Server)
Reconnecting with SMB1 for workgroup listing.
Server does not support EXTENDED_SECURITY  but 'client use spnego = yes' and 'client ntlmv2 auth = yes' is set
Anonymous login successful

	Server               Comment
	---------            -------
	KIOPTRIX             Samba Server

	Workgroup            Master
	---------            -------
	MYGROUP              KIOPTRIX
┌──[hexadivine@linux]─[~]
└──╼ $ smbclient \\\\192.168.0.102\\IPC$
Server does not support EXTENDED_SECURITY  but 'client use spnego = yes' and 'client ntlmv2 auth = yes' is set
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_NETWORK_ACCESS_DENIED listing \*
```