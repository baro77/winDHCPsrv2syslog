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
where Server_1 to Server_N are hosts or devices providing logs via syslog (usually network or Linux stuff), while Server_N+1 to Server_N+M are boxes providing their data to ELK by other means (for example Microsoft Active Directory Domain Controllers via https://www.elastic.co/downloads/beats/winlogbeat or Aruba Clearpass via https://github.com/njohnsn/ClearPassAndELK - thanks Neil :+1:).

Regarding DHCP, I have both Linux running dhcpd and Windows Server 2016 Standard **Core** running dhcp service; of course syslog was the natural choice for dhcpd, and I wanted to have the same "leases experience" coming from Microsoft world: so I have used an instance of NXLog Community Edition (https://nxlog.co/products/nxlog-community-edition) installed on Windows DHCP Server to mold syslog messages from ```c:\windows\system32\dhcp\DhcpSrvLog-*.log```.

The work consisted of two steps:
1. installing NXLog on Windows Server Core.
2. writing the configuration file to make NXLog act as the desired agent

## NXLog installation on Windows Server Core

TODO 

## Configuration file for NXLog

It's provided in this repository: https://github.com/baro77/winDHCPsrv2syslog/blob/master/nxlog.conf.

Just a few comments:
1. the configuration is written to substitute the standard configuration file provided with NXLog (in ```C:\Program Files (x86)\nxlog\conf``` if you are using the common settings)
2. it only handles IPv4 leases, but it shouldn't be difficult to add IPv6  (check it out ```DhcpSrvLog-*.log``` VS ```DhcpV6SrvLog-*.log``` files :wink:)
3. Windows leases have a bunch of trash at the top of the file explaining you how to read the actual lease lines: to avoid it the agent ignores all the lines not beginning with two digits and a comma (```if $raw_event !~ /^\d\d,.+/ {``` ...)
4. to be accurate, the timestamp of syslog messages originates from the actual date and time of the lease, not from the time the agent reads the lease
5. change the syslog facility and severity as you like, and don't forget to insert your remote syslogger address where you find ```# your syslog address here!!!``` comment
6. the lease is then written as-is in the MSG part of syslog message (so, for example, you'll find the date and time fields from which the timestamp has been generated): that's my choice because I just want a syslog message (I need to collect "old-school" textual logs in centralized syslogger, and delegate value-added parsing to ELK), but of course you have all you need to get more structured and powerful output since the agent CSV-analyze the lease line
7. Once in a PowerShell Session on the Windows Server (how-to in previous section), you can stop and start NXLog service (needed to reread conf file after changes) from the CLI with: ```Stop-Service -name nxlog``` and ```Start-Service -name nxlog```

