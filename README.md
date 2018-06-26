# winDHCPsrv2syslog
I'm implementing an ELK Stack for my employer with the following topology:

```
Server 1 ------+
               |
Server 2 ------+
               +-------- NXLog centralizes syslogger ------- ELK Server
...            |
               | 
Server n ------+

Server n+1 --------------------------------------------------------+

Server n+2 --------------------------------------------------------+

...

Server m+1 --------------------------------------------------------+
```
