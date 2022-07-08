# BF2556X-1T-ONIE-freeRouter
freeRouter ONIE install log

My notes on installing freeRouter on a BF2556X-1T

Upgraded BIOS from 1.0.0 to 1.2.1 to 1.3. Might be able to go direct. 
APS-Networks says to use option F when upgrading to 1.3

Boots into ONIE from new
ONIE-Install
onie-stop
ip addr add x.x.x.x/x dev eth2
ip route add default via x.x.x.x

Wait for ONIE to somehow magically start a dns resolver. /etc/resolv.conf - appears from somehere with 8.8.8.8
onie-nos-install 
onie-nos-install http://hydra.nix.net.switch.ch/RARE/releases/1/onie-installer.bin

Reboot

End up with a Debian prompt
Username: root/no password or rare/rare and rare is in suders. 

http://docs.freertr.org/guides/installation/wedge100bf32/rare-nos-lcm-cli/

Advised by Csaba to use tna-install-experimental
rare#show p4lang p4
category    value
peer        127.0.0.1
closed      0
since       2022-07-08 02:41:31
for         15:52:59
capability  mpls
platform    tna/bfforwarder
cpuport     -3
dynamicid   512 1023

front  name
0      frontpanel-1/0
13     frontpanel-10/0
14     frontpanel-11/0
15     frontpanel-12/0
28     frontpanel-13/0
29     frontpanel-14/0
30     frontpanel-15/0
31     frontpanel-16/0
44     frontpanel-17/0

rare#tna?
  tna-bfshell              - Start bfshell
  tna-cleanup              - Permanently remove unused packages
  tna-install-experimental - Install the latest commit of freerouter on top of the latest RARE development version
  tna-install-git          - Install the version from a specific Git commit
  tna-install-latest       - Install the latest RARE development version
  tna-install-release      - Install the specified release
  tna-list-available       - List all available releases
  tna-list-installed       - List all currently installed releases
  tna-list-long-installed  - List all currently installed releases with non-abbreviated Git tags
  tna-list-profiles        - List the available p4 profiles
  tna-set-profile          - Set the p4 profile and restart the data-plane processes
  tna-switch-to-generation - Select the RARE profile generation to activate and restart freerouter
  tna-uninstall-generation - Delete the specified generation, use tna-cleanup to permanently remove packages
  tna-update-release       - Install the latest update of the specified release


eth1 is the management port. 
Disable dhcp and statically configure a public IP address 
rare#show running-config int eth1
interface ethernet1
 description out of band management port
 vrf forwarding oob
 ipv4 address 193.1.249.200 255.255.255.192
 ipv6 address dynamic dynamic
 ipv6 gateway-prefix all6
 ipv6 slaac-client enable
 ipv6 prefix-suppress
 no shutdown
 no log-link-change
 exit
!

Configure static defauklt route
rare(cfg)#ipv4 route oob 0.0.0.0 0.0.0.0 193.1.249.193
rare(cfg)#exit
rare#ping 1.1.1.1 vrf oob
pinging 1.1.1.1, src=null, vrf=oob, cnt=5, len=64, tim=1000, gap=0, ttl=255, tos=0, sgt=0, flow=0, fill=0, sweep=false, multi=false

rare#ping www.cisco.com vrf oob
resolving www.cisco.com for ipv4 ok!
pinging 23.214.116.36, src=null, vrf=oob, cnt=5, len=64, tim=1000, gap=0, ttl=255, tos=0, sgt=0, flow=0, fill=0, sweep=false, multi=false
!!!!!
result=100%, recv/sent/lost/err=5/5/0/0, rtt min/avg/max/sum=12/12/12/61, ttl min/avg/max=53/53/53, tos min/avg/max=0/0/0
rare#tna-?
  tna-bfshell              - Start bfshell
  tna-cleanup              - Permanently remove unused packages
  tna-install-experimental - Install the latest commit of freerouter on top of the latest RARE development version
  tna-install-git          - Install the version from a specific Git commit
  tna-install-latest       - Install the latest RARE development version
  tna-install-release      - Install the specified release
  tna-list-available       - List all available releases
  tna-list-installed       - List all currently installed releases
  tna-list-long-installed  - List all currently installed releases with non-abbreviated Git tags
  tna-list-profiles        - List the available p4 profiles
  tna-set-profile          - Set the p4 profile and restart the data-plane processes
  tna-switch-to-generation - Select the RARE profile generation to activate and restart freerouter
  tna-uninstall-generation - Delete the specified generation, use tna-cleanup to permanently remove packages
  tna-update-release       - Install the latest update of the specified release

rare#tna-list-installed
Generation Current Release Git Tag                    KernelID       Kernel Release            Platform                   Install date
-----------------------------------------------------------------------------------------------------------------------------------------------------------
         1 *       1       release-1                  Debian11_0     5.10.0-8-amd64            stordis_bf2556x_1t         2022-07-08 01:00:18.258226175 +0200


rare#tna-list-available
INFO: Checking for release tags of https://bitbucket.software.geant.org/scm/rare/rare-nix.git
Unexpected error, aborting


rare#tna-install-experimental
child 14392 created on 3
child started
Unexpected error, aborting

process exited with 1536 code
% returned 6

After advice from Csaba - do a flash upgrade
rare#flash upgrade
 - downloading version info
tx:GET /rtr.ver HTTP/1.1
tx:User-Agent: freeRouter/22.4.23-cur
tx:Host: www.freertr.org
tx:Accept: */*
tx:Accept-Language: en,*
tx:Accept-Charset: iso-8859-1, *
tx:Accept-Encoding: identity
tx:Connection: Close
tx:
rx:HTTP/1.1 200 ok
rx:Server: freeRouter/22.7.7-cur
rx:Content-Type: text/plain
rx:Connection: Close
rx:Content-Length: 3169
rx:
 - receiving 3169 bytes
 * 3169 bytes done
 * old release: freeRouter v22.4.23-cur, done by cs@nop.
 * new release: freeRouter v22.7.7-cur, done by cs@nop.
 * old time: 1970-01-01 01:00:00
 * new time: 2022-07-07 21:20:53
 * diff: never
 * old files:/nix/store/2i5lj8gcjg68qxl4cglinnkci6wxsjj5-freerouter-jar-22.4.23/rtr.ver  /nix/store/2i5lj8gcjg68qxl4cglinnkci6wxsjj5-freerouter-jar-22.4.23/rtr.jar
 * new files:/nix/store/2i5lj8gcjg68qxl4cglinnkci6wxsjj5-freerouter-jar-22.4.23/rtr.ver rtr-x86_64.tar rtr-aarch64.tar rtr-native.sh rtr-library.sh rtr-pre.scr /nix/store/2i5lj8gcjg68qxl4cglinnkci6wxsjj5-freerouter-jar-22.4.23/rtr.jar rtr.zip rtr-post.scr
 * extra files:rtr-x86_64.tar rtr-aarch64.tar rtr-native.sh rtr-library.sh rtr-pre.scr rtr.zip rtr-post.scr
 * excess files:
 - downloading /etc/freertr/rtr-x86_64.tar
tx:GET /rtr-x86_64.tar HTTP/1.1
tx:User-Agent: freeRouter/22.4.23-cur
tx:Host: www.freertr.org
tx:Accept: */*
tx:Accept-Language: en,*
tx:Accept-Charset: iso-8859-1, *
tx:Accept-Encoding: identity
tx:Connection: Close
tx:
rx:HTTP/1.1 200 ok
rx:Server: freeRouter/22.7.7-cur
rx:Content-Type: application/tar
rx:Connection: Close
rx:Content-Length: 1300480
rx:
 - receiving 1300480 bytes
 * 1300480 bytes done
 - upgrading /etc/freertr/rtr-x86_64.tar
 - downloading /etc/freertr/rtr-aarch64.tar
tx:GET /rtr-aarch64.tar HTTP/1.1
tx:User-Agent: freeRouter/22.4.23-cur
tx:Host: www.freertr.org
tx:Accept: */*
tx:Accept-Language: en,*
tx:Accept-Charset: iso-8859-1, *
tx:Accept-Encoding: identity
tx:Connection: Close
tx:
rx:HTTP/1.1 200 ok
rx:Server: freeRouter/22.7.7-cur
rx:Content-Type: application/tar
rx:Connection: Close
rx:Content-Length: 2232320
rx:
 - receiving 2232320 bytes
 * 2232320 bytes done
 - upgrading /etc/freertr/rtr-aarch64.tar
 - downloading /etc/freertr/rtr-native.sh
tx:GET /rtr-native.sh HTTP/1.1
tx:User-Agent: freeRouter/22.4.23-cur
tx:Host: www.freertr.org
tx:Accept: */*
tx:Accept-Language: en,*
tx:Accept-Charset: iso-8859-1, *
tx:Accept-Encoding: identity
tx:Connection: Close
tx:
rx:HTTP/1.1 200 ok
rx:Server: freeRouter/22.7.7-cur
rx:Content-Type: text/plain
rx:Connection: Close
rx:Content-Length: 266
rx:
 - receiving 266 bytes
 * 266 bytes done
 - upgrading /etc/freertr/rtr-native.sh
 - downloading /etc/freertr/rtr-library.sh
tx:GET /rtr-library.sh HTTP/1.1
tx:User-Agent: freeRouter/22.4.23-cur
tx:Host: www.freertr.org
tx:Accept: */*
tx:Accept-Language: en,*
tx:Accept-Charset: iso-8859-1, *
tx:Accept-Encoding: identity
tx:Connection: Close
tx:
rx:HTTP/1.1 200 ok
rx:Server: freeRouter/22.7.7-cur
rx:Content-Type: text/plain
rx:Connection: Close
rx:Content-Length: 125
rx:
 - receiving 125 bytes
 * 125 bytes done
 - upgrading /etc/freertr/rtr-library.sh
 - downloading /etc/freertr/rtr-pre.scr
tx:GET /rtr-pre.scr HTTP/1.1
tx:User-Agent: freeRouter/22.4.23-cur
tx:Host: www.freertr.org
tx:Accept: */*
tx:Accept-Language: en,*
tx:Accept-Charset: iso-8859-1, *
tx:Accept-Encoding: identity
tx:Connection: Close
tx:
rx:HTTP/1.1 200 ok
rx:Server: freeRouter/22.7.7-cur
rx:Content-Type: text/plain
rx:Connection: Close
rx:Content-Length: 269
rx:
 - receiving 269 bytes
 * 269 bytes done
 - upgrading /etc/freertr/rtr-pre.scr
 * running upgrade script
 - downloading /nix/store/2i5lj8gcjg68qxl4cglinnkci6wxsjj5-freerouter-jar-22.4.23/rtr.jar
 - upgrading /nix/store/2i5lj8gcjg68qxl4cglinnkci6wxsjj5-freerouter-jar-22.4.23/rtr.jar
 * checksum mismatch, aborting!
rare#







