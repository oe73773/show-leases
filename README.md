# show-leases
show-leases is written in perl.

After deploying Internet Systems Consortium DHCP Server 4.3.3 (isc-dhcp) I was looking for a solution to get a quick overview about the used DHCP-Pool.

After playing around with different tools I decided to use the tool posted by Marcin Gosiewski posted on AskUbuntu https://askubuntu.com/questions/265504/how-to-monitor-dhcp-leased-ip-address/892586 as base.

I needed a tool to be used in the shell. So I changed the output to formated text. 
The origninal tool uses data from log and leasedb. I added a parser for dhcpd.conf. This will not only give information about Fixed Leases used in the logfile periode but about all configured leases. 

Because I only use IPv4 the tool supports IPv4 only. If some can provide Config, Logs and a leases DB for IPv6 or I find time to setup a test environment with IPv6 I will add IPv6 support.

I use the tool with Internet Systems Consortium DHCP Server 4.3.3 on Ubuntu 16.04.2 LTS.
Ubuntu 16.04.2 LTS uses traditional timestamp format by default. This is incomplete! Year and timezone is missing. 
e.g.
```
Apr 12 06:51:23 <hostname> dhcpd ....
```

I recommend to change the timestamp format in /etc/rsyslog.conf 
```
#
# Use traditional timestamp format.
# To enable high precision timestamps, comment out the following line.
#
#$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
```

After restart of rsyslog service timestamp in log is:
```
2017-04-19T06:47:07.226422+02:00 <hostname> dhcpd
```
For traditional timestamp I add the year. 
I assume the timezone for syslog is the same as the timezone the tool is using.

Depending on our setup you might need to change the files used (line 33-36):
```perl
my @logfilenames = ( "/var/log/syslog.1", "/var/log/syslog" );
my $leasedbname = "/var/lib/dhcp/dhcpd.leases";
my $configfilename = "/etc/dhcp/dhcpd.conf";
```

The syslog is parsed with regex. The regex are configured in line 38 - 53
```perl
#####################################################################################
# adjust regex your needs
#####################################################################################
my $IP4re = qr/\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}/;
my $MACre = qr/(?:[0-9A-Fa-f]{2}[:-]){5}(?:[0-9A-Fa-f]{2})/;
my $PROCre = qr/dhcpd\[\d*]: DHCPACK/;

  ###############################
  # Modify the following patterns to make regexp capture 5 groups from log line:
  # 1 - date
  # 2 - time
  # 3 - ip
  # 4 - mac
  # 5 - interface
my @LOGre = (   qr/(^[\d\-]{10})T(\d{2}:\d{2}:\d{2})\.[^\s]+\s.+\ ${PROCre} on (${IP4re}) to (${MACre}) (?:.*)via (.+)/,
                qr/^(.{3}(?:  \d| \d{2})) (\d{2}:\d{2}:\d{2}) .+ ${PROCre} on (${IP4re}) to (${MACre}) (?:.*)via (.+)/ );
```

The $PROCre regex is used in the LOG file pattern (@LOGre) but also in the code to preselect only DHCPACK lines for parsing (line 197).  
