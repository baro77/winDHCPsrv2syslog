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
where Server_1 to Server_N are hosts or devices providing logs via syslog (usually network or Linux stuff), while Server_N+1 to Server_N+M are boxes providing their data to ELK by other means (for example Microsoft Active Directory Domain Controllers via [Winlogbeat](https://www.elastic.co/downloads/beats/winlogbeat) or Aruba Clearpass via [Neil Johnson's CPPM & Logstash Configuration Files](https://github.com/njohnsn/ClearPassAndELK) - thanks Neil :+1:).

Regarding DHCP, I have both Linux running dhcpd and Windows Server 2016 Standard **Core** running dhcp service; of course syslog was the natural choice for dhcpd, and I wanted to have the same "leases experience" coming from Microsoft world: so I have used an instance of [NXLog Community Edition](https://nxlog.co/products/nxlog-community-edition) installed on Windows DHCP Server to mold syslog messages from ```c:\windows\system32\dhcp\DhcpSrvLog-*.log```.

The work consisted of two steps:
1. installing NXLog on Windows Server Core.
2. writing the configuration file to make NXLog act as the desired agent

## NXLog installation on Windows Server Core

I have used PowerShell over Windows Remote Management (WSMan), so I guess you could use the same approach if you have a **Nano** server:
1. on a Windows 10 workstation with Windows Remote Management service (WSMan) running, open PowerShell ISE **as Administrator**
2. if needed, trust your remote Windows DHCP Server: ```Set-Item WSMan:\LocalHost\Client\TrustedHosts "192.168.0.1"``` (let's suppose here and later that 192.168.0.1 is your Windows Server IP)
3. establish a session: ```$s = New-PSSession -ComputerName "192.168.0.1" -Credential ~\Administrator``` (provide remote server's Administrator password when prompted)
4. Copy the NXLog installer from your workstation to remote server (Administrator's Downloads folder is a good destination choice I guess): ```Copy-Item -ToSession $s -Path C:\Users\baro\Downloads\nxlog-ce-2.10.2102.msi -Destination C:\Users\Administrator\Downloads\nxlog-ce-2.10.2102.msi```
5. it's now time to switch to remote prompt: ```Enter-PSSession -Session $s```
6. "cd" to the folder where you copied NXLog installer at point 4 and launch **quiet** installation: ```msiexec /i nxlog-ce-2.10.2102.msi /quiet``` (usual NXLog installation is graphical, but no GUI here :wink:)
7. you can check NXLog service is installed and stopped with: ```Get-wmiobject win32_service | where Name -eq nxlog```
8. "cd" to NXLog standard configuration folder (remember you have done a standard quite installation): ```cd 'C:\Program Files (x86)\nxlog\conf'```
9. edit the configuration file: ```PSEdit .\nxlog.conf```, delete existing content, copy&paste content of the file provided in this repository (https://github.com/baro77/winDHCPsrv2syslog/blob/master/nxlog.conf), and  save the file (NOTE: instead of editing of course you can copy the configuration file as you have done with NXLog installer, if you prefer)
10. you can stop and start NXLog service (needed to reread conf file after every new changes) with: ```Stop-Service -name nxlog``` and ```Start-Service -name nxlog```
11. Once started, you can check NXLog's logs in ```C:\Program Files (x86)\nxlog\data\nxlog.log```
12. When all is done, you can exit from PowerShell remote prompt with ```Exit-PSSession```

## Configuration file for NXLog

It's provided in this repository: https://github.com/baro77/winDHCPsrv2syslog/blob/master/nxlog.conf.

Just a few comments:
1. the configuration is written to substitute the standard configuration file provided with NXLog (in ```C:\Program Files (x86)\nxlog\conf``` if you are using the common settings)
2. it only handles IPv4 leases, but it shouldn't be difficult to add IPv6  (check it out ```DhcpSrvLog-*.log``` VS ```DhcpV6SrvLog-*.log``` files :wink:)
3. Windows leases have a bunch of trash at the top of the file explaining you how to read the actual lease lines: to avoid it the agent ignores all the lines not beginning with two digits and a comma (```if $raw_event !~ /^\d\d,.+/ {``` ...)
4. to be accurate, the timestamp of syslog messages originates from the actual date and time of the lease, not from the time the agent reads the lease
5. change the syslog facility and severity as you like, and don't forget to insert your remote syslogger address where you find ```# your syslog address here!!!``` comment
6. the lease is then written as-is in the MSG part of syslog message (so, for example, you'll find the date and time fields from which the timestamp has been generated): that's my choice because I just want a syslog message (I need to collect "old-school" textual logs in centralized syslogger, and delegate value-added parsing to ELK), but of course you have all you need to get more structured and powerful output since the agent CSV-analyze the lease line


