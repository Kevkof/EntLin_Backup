# Enterprise Linux Lab Report - Troubleshooting

- Student name: `Kevin Verlinde`
- Class/group: TIN-TI-3B1 (Gent)

## Instructions

- Write a detailed report in the "Report" section below (in Dutch or English)
- Use correct Markdown! Use fenced code blocks for commands and their output, terminal transcripts, ...
- The different phases in the bottom-up troubleshooting process are described in their own subsections (heading of level 3, i.e. starting with `###`) with the name of the phase as title.
- Every step is described in detail:
    - describe what is being tested
    - give the command, including options and arguments, needed to execute the test, or the absolute path to the configuration file to be verified
    - give the expected output of the command or content of the configuration file (only the relevant parts are sufficient!)
    - if the actual output is different from the one expected, explain the cause and describe how you fixed this by giving the exact commands or necessary changes to configuration files
- In the section "End result", describe the final state of the service:
    - copy/paste a transcript of running the acceptance tests
    - describe the result of accessing the service from the host system
    - describe any error messages that still remain

## Report

Setting the systeminput to azerty:
```
sudo localectl set-keymap be
sudo localectl set-x11-keymap be
sudo loadkeys fr
```
First two commands were for belgian azerty, my laptop is in french azerty so the last command is added

### Phase 1: Link Layer
- The vm is connected to a host-only adapter with the ip: `172.16.128.1`
  - Solution: Connect vm to host-only with ip: `192.168.56.1`
- Interface `enp0s8` is down
  - Solution: `sudo ip link set enp0s8 up`


### Phase 2: Internet Layer
- Checking ip assignments with `ip a`:
  - [x] enp0s3: ip 10.0.2.15/24
  - [ ] enp0s8: ip 192.168.56.42/24
    - Edit the ifcfg for the interface (located in: `/etc/sysconfig/network-scripts`)

    ```
    #VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
NM_CONTROLLED=no
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.56.42
NETMASK=255.255.255.0
DEVICE=enp0s8
GATEWAY=192.168.56.1
TYPE=Ethernet
PEERDNS=no
#VAGRANT-END
    ```
    
- Rechecking ip assignments with `ip a`
  - [x] enp0s3: ip 10.0.2.15/24
  - [x] enp0s8: ip 192.168.56.42/24

- Checking Default Gateway with `ip r`
  - [x] default via 10.0.2.2 dev enp0s3
  - [x] default via 192.168.56.1 dev enp0s8

- Checking DNS-server with `cat /etc/resolv.conf`
  - [x] nameserver 10.0.2.3

  - Lan Connectivity:
    - [x] Ping Default Gateway `ping -c 4 10.0.2.2` (`-c 4`, this is to automatically stop the pings after 4 attempts)
    - [x] Ping Default Gateway `ping -c 4 192.168.56.1` (`-c 4`, this is to automatically stop the pings after 4 attempts)
    - [x] Ping DNS server `ping -c 4 10.0.2.3` (`-c 4`, this is to automatically stop the pings after 4 attempts)
  
### Phase 3: Transport Layer & Application Layer
- Check if bind is running with `systemctl status named`
  - Output:
  ```
  named.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named.service; enabled)
   Active: failed (Result: exit-code) since Fri 2017-12-08 09:27:15 UTC; 5min ago
  Process: 1305 ExecStartPre=/usr/sbin/named-checkconf -z /etc/named.conf (code=exited, status=1/FAILURE)
  ```
  - This most likely means the syntax of the named.conf file is wrong
    - Add acl for the networks mentioned in ~/README.md
    ```
    192.168.56/24
    192.0.2/24
    ```
    - Changed yes to True
    ```
    dnssec-enable yes;      ==> dnssec-enable True;
    dnssec-validation yes;  ==> dnssec-validation True;
    ```
  - Still unable to start the named service
  Output: 
  ```
  named.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named.service; enabled)
   Active: failed (Result: exit-code) since Fri 2017-12-08 10:01:48 UTC; 5min ago
  Process: 2758 ExecStartPre=/usr/sbin/named-checkconf -z /etc/named.conf (code=exited, status=1/FAILURE)

Dec 08 10:01:48 golbat.cynalco.com systemd[1]: Starting Berkeley Internet Name Domain (DNS)...
Dec 08 10:01:48 golbat.cynalco.com named-checkconf[2758]: zone cynalco.com/IN: loaded serial 15081921
Dec 08 10:01:48 golbat.cynalco.com named-checkconf[2758]: zone 192.168.56.in-addr.arpa/IN: loaded serial 15081921
Dec 08 10:01:48 golbat.cynalco.com named-checkconf[2758]: zone 2.0.192.in-addr.arpa/IN: NS 'golbat.cynalco.com.2.0.192.in-addr.arpa' has no address records (A or AAAA)
Dec 08 10:01:48 golbat.cynalco.com named-checkconf[2758]: zone 2.0.192.in-addr.arpa/IN: not loaded due to errors.
Dec 08 10:01:48 golbat.cynalco.com named-checkconf[2758]: _default/2.0.192.in-addr.arpa/IN: bad zone
Dec 08 10:01:48 golbat.cynalco.com systemd[1]: named.service: control process exited, code=exited status=1
Dec 08 10:01:48 golbat.cynalco.com systemd[1]: Failed to start Berkeley Internet Name Domain (DNS).
Dec 08 10:01:48 golbat.cynalco.com systemd[1]: Unit named.service entered failed state.
  ```
  - Login as root using `su -` to access the directory `/var/named`
    - In `cynalco.com` add CNAME record for mankey
## End result



## Resources

List all sources of useful information that you encountered while completing this assignment: books, manuals, HOWTO's, blog posts, etc.