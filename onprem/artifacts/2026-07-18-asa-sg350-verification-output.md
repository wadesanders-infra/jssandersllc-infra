# ASA and SG350 Console Output: WS1 Verification Session

**Session date:** 2026-07-18 (clock and NTP re-checked live 2026-07-19)
**Capture method:** Verbatim console output captured from the terminal during the session. Automated session logging was not running, so this record was compiled from the terminal capture rather than written by PuTTY; the command output itself is unaltered. Windows-side output (DC01, SRV01, JSS-WS01, JSS-WS02) was captured to per-host files.
**Sanitization:** Reviewed; no credentials, hashes, or secrets appear in any output.

## ASA: Phase 1 (2026-07-18)

```
JSS-ASA01# terminal pager 0
JSS-ASA01# show nameif
Interface                Name                     Security
GigabitEthernet1/1       outside                    0
GigabitEthernet1/2.10    MGMT                      90
GigabitEthernet1/2.20    SERVERS                   80
GigabitEthernet1/2.30    CLIENTS                   70
GigabitEthernet1/2.50    USER                      60
JSS-ASA01# show running-config interface
!
interface GigabitEthernet1/1
 nameif outside
 security-level 0
 ip address dhcp setroute
!
interface GigabitEthernet1/2
 no nameif
 no security-level
 no ip address
!
interface GigabitEthernet1/2.10
 vlan 10
 nameif MGMT
 security-level 90
 ip address 10.10.10.1 255.255.255.0
!
interface GigabitEthernet1/2.20
 vlan 20
 nameif SERVERS
 security-level 80
 ip address 10.10.20.1 255.255.255.0
!
interface GigabitEthernet1/2.30
 vlan 30
 nameif CLIENTS
 security-level 70
 ip address 10.10.30.1 255.255.255.0
!
interface GigabitEthernet1/2.50
 vlan 50
 nameif USER
 security-level 60
 ip address 10.10.50.1 255.255.255.0
!
interface GigabitEthernet1/3
 shutdown
 no nameif
 no security-level
 no ip address
!
interface GigabitEthernet1/4
 shutdown
 no nameif
 no security-level
 no ip address
!
interface GigabitEthernet1/5
 shutdown
 no nameif
 no security-level
 no ip address
!
interface GigabitEthernet1/6
 shutdown
 no nameif
 no security-level
 no ip address
!
interface GigabitEthernet1/7
 shutdown
 no nameif
 no security-level
 no ip address
!
interface GigabitEthernet1/8
 shutdown
 no nameif
 no security-level
 no ip address
!
interface Management1/1
 management-only
 shutdown
 no nameif
 no security-level
 no ip address
JSS-ASA01# show running-config access-list
access-list USER_IN extended permit udp 10.10.50.0 255.255.255.0 host 10.10.20.10 eq domain
access-list USER_IN extended permit tcp 10.10.50.0 255.255.255.0 host 10.10.20.10 eq domain
access-list USER_IN extended permit ip 10.10.50.0 255.255.255.0 any
access-list CLIENTS_IN extended permit udp 10.10.30.0 255.255.255.0 host 10.10.20.10 eq domain
access-list CLIENTS_IN extended permit tcp 10.10.30.0 255.255.255.0 host 10.10.20.10 eq domain
access-list CLIENTS_IN extended permit ip 10.10.30.0 255.255.255.0 any
JSS-ASA01# show running-config access-group
access-group CLIENTS_IN in interface CLIENTS
access-group USER_IN in interface USER
JSS-ASA01# show running-config dhcprelay
dhcprelay server 10.10.20.10 SERVERS
dhcprelay enable CLIENTS
dhcprelay enable USER
dhcprelay timeout 60
JSS-ASA01# show running-config nat
!
object network MGMT_NAT
 nat (MGMT,outside) dynamic interface
object network SERVERS_NAT
 nat (SERVERS,outside) dynamic interface
object network CLIENTS_NAT
 nat (CLIENTS,outside) dynamic interface
object network USER_NAT
 nat (USER,outside) dynamic interface
```

## ASA: Phase 5 clock and NTP

First check, 2026-07-18 (transcribed):

```
JSS-ASA01# show clock
19:49:49.564 UTC Sat Jul 18 2026
JSS-ASA01# show running-config ntp
JSS-ASA01#
```

Re-check, 2026-07-19:

```
JSS-ASA01# terminal pager 0
JSS-ASA01# show clock
10:28:24.893 UTC Sun Jul 19 2026
JSS-ASA01# show running-config ntp
JSS-ASA01#
```

Both checks: clock correct, NTP unconfigured (finding F7).

## SG350: step 1.7

First capture, 2026-07-18, laptops powered off:

```
switchd5be16#show mac address-table
Flags: I - Internal usage VLAN
Aging time is 300 sec
    Vlan          Mac Address         Port       Type
------------ --------------------- ---------- ----------
     1         34:17:eb:b1:b8:88      gi3      dynamic
     1         3c:57:31:d5:be:16       0         self
     10        34:17:eb:b1:b8:88      gi3      dynamic
     10        68:3b:78:aa:7b:2e      gi1      dynamic
     20        00:15:5d:01:85:01      gi3      dynamic
     20        00:15:5d:01:85:02      gi3      dynamic
     20        68:3b:78:aa:7b:2e      gi1      dynamic
     50        68:3b:78:aa:7b:2e      gi1      dynamic
     50        d8:bb:c1:da:cc:3a      gi2      dynamic
```

Second capture, 2026-07-18, after booting both laptops (switch's own log lines included; note the 2019 timestamps, finding F6):

```
switchd5be16#26-Jun-2019 04:23:30 %LINK-I-Up:  gi7
26-Jun-2019 04:23:35 %STP-W-PORTSTATUS: gi7: STP status Forwarding
26-Jun-2019 04:23:49 %LINK-I-Up:  gi8
26-Jun-2019 04:23:54 %STP-W-PORTSTATUS: gi8: STP status Forwarding
show mac address-table
Flags: I - Internal usage VLAN
Aging time is 300 sec
    Vlan          Mac Address         Port       Type
------------ --------------------- ---------- ----------
     1         34:17:eb:b1:b8:88      gi3      dynamic
     1         3c:57:31:d5:be:16       0         self
     10        34:17:eb:b1:b8:88      gi3      dynamic
     10        68:3b:78:aa:7b:2e      gi1      dynamic
     20        00:15:5d:01:85:01      gi3      dynamic
     20        00:15:5d:01:85:02      gi3      dynamic
     20        68:3b:78:aa:7b:2e      gi1      dynamic
     30        00:21:70:e9:0c:b8      gi8      dynamic
     30        20:7b:d2:be:4a:5d      gi7      dynamic
     30        68:3b:78:aa:7b:2e      gi1      dynamic
     50        68:3b:78:aa:7b:2e      gi1      dynamic
     50        d8:bb:c1:da:cc:3a      gi2      dynamic
```

The factory hostname visible in the prompt is finding F2; the switch's presence on VLAN 1 only (self entry) is finding F3.
