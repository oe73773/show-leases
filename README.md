# show-leases
show-leases is written in perl.

After deploying Internet Systems Consortium DHCP Server 4.3.3 I was looking for a solution to get a quick overview about the used DHCP-Pool.

After playing around with different tools I decided to use the tool posted by Marcin Gosiewski posted on AskUbuntu https://askubuntu.com/questions/265504/how-to-monitor-dhcp-leased-ip-address/892586 

I change the perl code to produce a easy readable shell output.
The origninal tool uses data from log and leasedb.
I added a parser for dhcpd.conf.

I use the tool with Internet Systems Consortium DHCP Server 4.3.3 on Ubuntu 16.04.2 LTS.
Depending on our setup you might need to change the files used (line 30-32):

my @logfilenames = ( "/var/log/syslog.1", "/var/log/syslog" );
my $leasedbname = "/var/lib/dhcp/dhcpd.leases";
my $configfilename = "/etc/dhcp/dhcpd.conf";

The syslog is parsed with regex in line 106 and 107 and might need to be change to fit your log format.
