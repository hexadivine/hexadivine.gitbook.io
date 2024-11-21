![](Pasted%20image%2020241121153341.png)
## Introduction

Burp suit is used to test web & mobile apps. It allows us to view, and edit http/https traffic.
Some features include:

- **Proxy:** Burp Proxy operates as a web proxy server between the browser and target applications. It enables you to intercept, inspect, and modify traffic that passes in both directions. You can even use this to test using HTTPS. 
- **Repeater:** Repeater allows for capturing, modifying, and re-sending the same request multiple times. This functionality is particularly useful when crafting payloads through trial and error (e.g., in SQLi - Structured Query Language Injection) or testing the functionality of an endpoint for vulnerabilities.
- **Intruder:** Burp Intruder is a tool for automating customised attacks against web applications. It enables you to configure attacks that send the same HTTP request over and over again, inserting different payloads into predefined positions each time. 
- 

## References

- https://portswigger.net/burp/documentation/desktop/tools/proxy
- https://portswigger.net/burp/documentation/desktop/tools/intruder
- https://tryhackme.com/r/room/burpsuitebasics