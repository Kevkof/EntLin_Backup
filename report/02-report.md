# Enterprise Linux Lab Report

- Student name: `Kevin Verlinde`
- Github repo: <https://github.com/HoGentTIN/elnx-sme-KevinVerlinde.git>

- Being able to set up and test a DNS-server with BIND
   - Understanding BIND's configuration files, particularly zone files, and being able to find and fix errors
   - Knowing how to query a DNS server with `dig`

## Test plan

### pu001 Master DNS

1. On the host system, go to the local working directory of the project repository
2. Execute `vagrant status`
    - The VM, `pu001` should have status `not created`. If the VM does exist, destroy it first with `vagrant destroy -f pu001`
3. Execute `vagrant up pu001`
    - The command should run without errors (exit status 0)
4. Log in on the server with `vagrant ssh pu001` and run the acceptance tests with `sudo /vagrant/tests/runbats.sh`. 

- 10 tests for the normal install (those have already been tested and documented in [00-report.md](00-report.md))
- 13 tests for the master DNS

5. Test if the dig command to returns the ip, we'll test this with `dig @<dns_server_ip> <host_name>.<domain> +short`. This should return the ip address of the server we're digging to. For our test we'll use the lamp stack that we made in the previous assignment for `dig @192.0.2.10 pu004.avalon.lan +short`, which should return `192.0.2.50`

### pu002 Slave DNS

1. On the host system, go to the local working directory of the project repository
2. Execute `vagrant status`
    - The VM, `pu002` should have status `not created`. If the VM does exist, destroy it first with `vagrant destroy -f pu002`.
    - The VM `pu001` has to be running so that the slave dns can copy the hosts from the master dns
3. Execute `vagrant up pu002`
    - The command should run without errors (exit status 0)
4. Log in on the server with `vagrant ssh pu002` and run the acceptance tests with `sudo /vagrant/tests/runbats.sh`. 

- 10 tests for the normal install (those have already been tested and documented in [00-report.md](00-report.md))
- 14 tests for the slave DNS

5. Test if the dig command to returns the ip, we'll test this with `dig @<dns_server_ip> <host_name>.<domain> +short`. This should return the ip address of the server we're digging to. For our test we'll use the lamp stack that we made in the previous assignment for `dig @192.0.2.11 pu004.avalon.lan +short`, which should return `192.0.2.50`


## Procedure/Documentation

#### vagrant-hosts.yml
[Link to file](../vagrant-hosts.yml)

- Add entries for pu001 and pu002 with their respective IP's

#### site.yml
[Link to file](../ansible/site.yml)

- Add entries for pu001 and pu002 with the `Bind` role

#### pu001.yml

- Add dns to the allowed firewall services
- Add bind parameters:
  - Access Control List
  - Zone Name
  - Master server ip
  - Iternal name servers
  - The networks it handles
  - The mail server
  - The hosts within the network
  - The address it's allowed to listen on
  - The hosts that are allowed to query the server

#### pu002.yml
  
- Add dns to the allowed firewall services
- Add bind parameters:
  - Access Control List
  - Zone Name
  - Master server ip
  - Iternal name servers
  - The networks it handles
  - The mail server
  - The address it's allowed to listen on
  - The hosts that are allowed to query the server

## Test report

[Evidence Video](https://youtu.be/KLNouGZyxqM)

### Pu001 Master DNS

1. `vagrant status` shows no VM's running.
2. `vagrant up pu001` start the boot process.  
3. `vagrant ssh pu001` to log into the vm
4. Running the acceptance tests with `sudo /vagrant/test/runbats.sh`

```
✓ The `dig` command should be installed
✓ The main config file should be syntactically correct
✓ The forward zone file should be syntactically correct
✓ The reverse zone files should be syntactically correct
✓ The service should be running
✓ Forward lookups public servers
✓ Forward lookups private servers
✓ Reverse lookups public servers
✓ Reverse lookups private servers
✓ Alias lookups public servers
✓ Alias lookups private servers
✓ NS record lookup
✓ Mail server lookup

13 tests, 0 failures
```

- [x] `dig @192.0.2.10 pu004.avalon.lan +short` results in `192.0.2.50`

### Pu002 Slave DNS

[Evidence Video](https://youtu.be/_aHy2gWG34k)

1. `vagrant status` shows only the `pu001` vm is running.
2. `vagrant up pu002` start the boot process.  
3. `vagrant ssh pu002` to log into the vm
4. Running the acceptance tests with `sudo /vagrant/test/runbats.sh`

```
✓ The `dig` command should be installed
✓ The main config file should be syntactically correct
✓ The server should be set up as a slave
✓ The server should forward requests to the master server
✓ There should not be a forward zone file
✓ The service should be running
✓ Forward lookups public servers
✓ Forward lookups private servers
✓ Reverse lookups public servers
✓ Reverse lookups private servers
✓ Alias lookups public servers
✓ Alias lookups private servers
✓ NS record lookup
✓ Mail server lookup

14 tests, 0 failures
```

- [x] `Dig @192.0.2.11 pu004.avalon.lan +short` results in `192.0.2.50`

## Resources

- [Ansible role Bind (by bertvv)](https://galaxy.ansible.com/bertvv/bind/)
- Files P3ops-Green
