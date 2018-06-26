# winDHCPsrv2syslog
I'm implementing an ELK Stack for my employer with the following topology:
```
Server_1 ------+
               |
Server_2 ------+
               +-------- centralized syslogger ------- ELK Server
...            |                                            |
               |                                            |
Server_N ------+                                            |
                                                            |
Server_N+1 -------------------------------------------------+
                                                            |
Server_N+2 -------------------------------------------------+
                                                            |
...                                                         |
                                                            |
Server_N+M -------------------------------------------------+
```
where Server_1 to Server_N are hosts or devices providing logs via syslog (usually network or Linux stuff), while Server_N+1 to Server_N+M are boxes providing their data to ELK by other means (for example Microsoft ADDC via https://www.elastic.co/downloads/beats/winlogbeat or Aruba Clearpass via https://github.com/njohnsn/ClearPassAndELK - thanks Neil :-) ).

Regarding DHCP, I have both Linux running dhcpd and Windows Server **Core Edition** running dhcp service; of course syslog was the natural choice for dhcpd, and I wanted to have the same "leases experience" coming from Microsoft world: so I have used an instance of NXLog Community Edition (https://nxlog.co/products/nxlog-community-edition) installed on Windows Server to implement a small agent to mold syslog messages from ```c:\windows\system32\dhcp\DhcpSrvLog-*.log```.

The work consisted of two steps:
1. installing NXLog on Windows Server Core Ed.
2. writing the configuration file to make NXLog act as the desired agent

## NXLog installation on Windows Server Core

TODO 

## Configuration file for NXLog

Its provided in 


