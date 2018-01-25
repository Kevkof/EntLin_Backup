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

### Router

Setting the systeminput to azerty: 
```
sudo dpkg-reconfigure keyboard-configuration
```

The machine needs to be restarted for the kayboard change to register

#### Phase 1: Link Layer
- The 'hardware' is connecting correctly (in virtualbox)
- Output for `ip link` shows that both interfaces are up

#### Phase 2: Internet Layer
- Checking ip assignments with `ip a`:
  - [x] eth0: ip 10.0.2.15/24
  - [ ] eth1: ip 172.20.0.254/24, this shows 172.20.0.254/8 instead
    - in configure mode use `delete interfaces ethernet eth1 address 172.20.0.254/8` to remove the old IP
    - use `set interfaces ethernet eth1 address 172.20.0.254/24` to add the new IP
    - then commit, save and exit
  - re-testing with `ip a`
  - [x] eth1: ip 172.20.0.254/24
- Checking Default Gateway with `ip r`
  - [x] default via 10.0.2.2 dev eth0 proto zebra
- Checking DNS-server with `cat /etc/resolv.conf`
  - [ ] nameserver 172.20.0.254, this is the router ip, not the ip for the server
  - [x] nameserver 10.0.2.3
    - set the dns forwarding using (in configure mode): 
    ```
    # delete system name-server 172.20.0.254
    # set service dns forwarding domain linuxlab.lan server 172.20.0.2
    # set service dns forwarding name-server 10.0.2.3
    # set service dns forwarding listen-on 'eth1'
    ```
    !! don't forget to commit, save and exit

### Server

Setting the systeminput to azerty:
```
sudo localectl set-keymap be
sudo localectl set-x11-keymap be
```

#### Phase 1: Link Layer
- The 'hardware' is connecting correctly (in virtualbox)
- Output for `ip link` shows that both interfaces are up

#### Phase 2: Internet Layer
- Checking ip assignments with `ip a`:
  - [x] enp0s3: ip 10.0.2.15/24
  - [x] enp0s8: ip 172.20.0.2/24
- Checking Default Gateway with `ip r`
  - [x] default via 10.0.2.2 dev enp0s3
- Checking DNS-server with `cat /etc/resolv.conf`
  - [x] nameserver 10.0.2.3, this is the DNS server from virtualbox

- Lan Connectivity:
  - [x] Ping Default Gateway `ping -c 4 10.0.2.2` (`-c 4`, this is to automatically stop the pings after 4 attempts)
  - [x] Ping DNS server `ping -c 4 10.0.2.3` (`-c 4`, this is to automatically stop the pings after 4 attempts)

#### Phase 3: Transport layer & Application layer
- Check if dnsmasq is running with `systemctl status dnsmasq`
  - Shows 'Active: inactive (dead)' so the service is not running
  - Attempting to start with `sudo systemctl start dnsmasq`
    - No errors
- Check if dhcp is running with `systemctl status dhcpd`
  - Shows 'Active: inactive (dead)' so the service is not running
  - Attempting to start with `sudo systemctl start dhcpd`
    - Returns the following error: 
    ```
    Job for dhcpd.service failed because the control process exited with error code. see "systemctl status dhcpd.service" and "journalctl -xe" for details.
    ```
  - The output for `systemctl status dhcpd` tells us there was a failure when trying to start the service (nothing new)
  - Looking for more information on the error with `journalctl -xe`, which tells us that there were no subnets declared for either network interface
    - In the dhcpd.conf file add `interface enp0s8` to allow a subnet to be set to that interface
    - Change the netmask to be `255.255.255.0` instead of `255.255.0.0`
    - Change the subnet and the range from `172.22.0.x` to `172.20.0.x`
    - Change the range to be from host 101 to host 253
    - Change the router from `10.0.2.2` to `172.20.0.254`
  - Start the service using `sudo systemctl start dhcpd`
- Check if Apache is running with `systemctl status httpd`
  - Shows 'Active: inactive (dead)' so the service is not running
  - Attempting to start with `sudo systemctl start httpd`
    - No errors, `systemctl status httpd` shows that the service is disabled so it won't start on system boot
    - Enable the service using `sudo systemctl enable httpd`
  - After tests on the workstation it seems the connection to the web content isn't possible, looking in the file `/etc/httpd/conf/httpd.conf`
    - At 'Listen 80' add a line with: `Listen 172.20.0.2:80`
- Using `sudo ss -tulpn` we can see what ports allow for traffic to pass through the firewall.
  - [x] ports 53 (DNS) and 80 (HTTP) are both open


### Workstation

Setting the systeminput to azerty:
```
sudo localectl set-keymap be
sudo localectl set-x11-keymap be
```

#### Phase 1: Link Layer
- Disconnect the cable for the NAT interface (following the assignment)
- Output for `ip link` looks good, enp0s8 (intnet) is up while enp0s3 (NAT) is down

#### Phase 2: Internet Layer
- Checking IP assignments with `ip a`
  - [x] Address in the dhcp range given by the server
- Checking Default Gateway with `ip r`
  - [x] default via 172.20.0.254 dev enp0s8 
- Checking DNS-Server with `cat /etc/resolv.conf`
  - [x] `search linuxlab.lan` and `nameserver 172.20.0.2`
- Ping tests: 
  - [x] Default gateway (172.20.0.254)
  - [x] Server (www.linuxlab.lan)
  - [x] Outside (icanhazip.com)

#### Phase 3: Transport layer & Application layer
- No specific services needed

## End result
```console
[vagrant@workstation ~]$ acceptance-tests
* The host IP address should be in the expected range
* The gateway from the NAT interface should not be active.
* The DNS service from the NAT interface should not be active.
* Name resolution for external websites should succeed
* Name resolution for internal websites should succeed
* It should be possible to access www.linuxlab.lan
* It should be possible to access websites on the internet

7 tests, 0 failures
```

## Resources

- [VyOS cheat sheet by bertvv](https://github.com/bertvv/cheat-sheets/blob/master/docs/VyOS.md)
- [Common administrative commands in Red Hat Enterprise Linux 5, 6, and 7](https://access.redhat.com/articles/1189123)
- [CentOS httpd listening only on IPv6 interface on CentOS 7](https://lists.centos.org/pipermail/centos/2014-December/148494.html)
