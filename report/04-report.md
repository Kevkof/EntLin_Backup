# Enterprise Linux Lab Report 04

- Student name: `Kevin Verlinde`
- Github repo: <https://github.com/HoGentTIN/elnx-sme-KevinVerlinde.git>

- Learning to work with
  - DHCP servers
  - DNS servers
  - Networking in VyOS

## Test plan

- Create a VM with 2 network adapters (same host-only adapter as the DHCP server)
  - Make sure that the mac address of 1 of them is the one in the DHCP config
- Boot this Linux VM
- Verify that:
  - Both interfaces have an IP assigned (one predetermined, 172.16.155.155, and one from the pool), done with `ifconfig`
  - The vm can browse to the 'site' on http://www.avalon.lan (/wordpress for the actual wordpress), this can be done from any browser
  - The vm can access the fileserver via smb (using nautilus) 
  - The vm can access the fileserver via ftp (using nautilus) 

## Procedure/Documentation

#### pr001.yml
[Link to file](../ansible/host_vars/pr001.yml)
- Add Global options for dhcp:
  - lease time
  - subnet mask
  - domain name
  - dns servers (both in the network and the router to pass outside of the network)
  - router address
- Add subnets:
  - Known hosts, ip range: 172.16.128.1 to 172.16.191.254
  - Unknown hosts, ip range: 172.16.192.1 to 172.16.255.253
- Add dhcp host
  - This has the mac address for one of the interfaces the test WS

#### router-config.sh
[Link to file](../scripts/router-config.sh)
- Add domain name
- Add interfaces (eth0, eth1 and eth2) with their respective ip settings
- Add NAT rules for eth0 and eth1
- Add DNS forwarding: 
  - within the domain to the domain dns
  - outside the domain to the virtualbox dns
  - interfaces to listen on

## Test report

- VM info: 

![HO1](https://i.imgur.com/oZNZRrs.png)  

Note the mac address (the same one as in [pr001.yml](../ansible/host_vars/pr001.yml))

![HO2](https://i.imgur.com/kfqFbQX.png)

- Verifications:

### enp0s3 has the IP from the pool and enp0s8 has the assigned IP

![ifconfig](https://i.imgur.com/wPpeTod.png)

### http://avalon.lan can be reached

![browser test](https://i.imgur.com/AXpGyNq.png)

### SMB and FTP to the fileshare

![before](https://i.imgur.com/eG1p7Sx.png)

![after](https://i.imgur.com/r08Dz2a.png)

## Resources
- Project files p3ops-green
- [VyOS cheat sheet by bertvv](https://github.com/bertvv/cheat-sheets/blob/master/docs/VyOS.md)
