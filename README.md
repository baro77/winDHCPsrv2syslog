# winDHCPsrv2syslog
I'm implementing an ELK Stack for my employer with the following topology:
```
Server_1 ------+
               |
Server_2 ------+
               +-------- NXLog centralized syslogger ------- ELK Server
...            |                                                  |
               |                                                  |
Server_N ------+                                                  |
                                                                  |
Server_N+1 -------------------------------------------------------+
                                                                  |
Server_N+2 -------------------------------------------------------+
                                                                  |
...                                                               |
                                                                  |
Server_N+M -------------------------------------------------------+
```
where Server_1 to Server_N are hosts or devices providing logs via syslog (usually network or Linux stuff or Linux), while Server_N+1 to Server_N+M are boxes providing their data to ELK by other means (for example Microsoft ADDC via https://www.elastic.co/downloads/beats/winlogbeat or Aruba Clearpass via great https://github.com/njohnsn/ClearPassAndELK)
