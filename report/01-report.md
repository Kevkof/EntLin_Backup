# Enterprise Linux Lab Report 01

- Student name: `Kevin Verlinde`
- Github repo: <https://github.com/HoGentTIN/elnx-sme-KevinVerlinde.git>

Purpose: Installing and configuring a LAMP stack

## Test plan

1. On the host system, go to the local working directory of the project repository
2. Execute `vagrant status`
    - The VM, `pu004` should have status `not created`. If the VM does exist, destroy it first with `vagrant destroy -f pu004`
3. Execute `vagrant up pu004`
    - The command should run without errors (exit status 0)
4. Log in on the server with `vagrant ssh pu004` and run the acceptance tests with `sudo /vagrant/tests/runbats.sh`. 

- 10 tests for the normal install (those have already been tested and documented in [00-report.md](00-report.md))
- 15 tests for the lamp stack

## Procedure/Documentation

#### site.yml

[Link to the file](../ansible/site.yml)

- Add roles to host pu004
  - httpd
  - mariadb
  - WordPress

#### host_vars/pu004.yml

[Link to the file](../ansible/host_vars/pu004.yml)

- Allow required firewall services
- MariaDB:
  - Root password
  - Database
  - User
- WordPress:
  - Database
  - User
  - Password
- Add post_tasks:
  - Copy the certificate

### Create the certificate

- openssl genrsa -out pu004.key 2048 
- openssl req -new -key pu004.key -out pu004.csr
- openssl x509 -req -days 365 -in pu004.csr -signkey pu004.key -out pu004.crt

## Test report

[Evidence Footage](https://www.youtube.com/watch?v=vdz7oEPOWSQ)

1. `vagrant status` shows no VM's running.
2. `vagrant up pu004` start the boot process.  
3. `vagrant ssh pu004` to log into the vm
4. Running the acceptance tests with `sudo /vagrant/test/runbats.sh`

Test results: 
```
Running test /vagrant/test/pu004/lamp.bats
 ✓ The necessary packages should be installed
 ✓ The Apache service should be running
 ✓ The Apache service should be started at boot
 ✓ The MariaDB service should be running
 ✓ The MariaDB service should be started at boot
 ✓ The SELinux status should be ‘enforcing’
 ✓ Web traffic should pass through the firewall
 ✓ Mariadb should have a database for Wordpress
 ✓ The MariaDB user should have "write access" to the database
 ✓ The website should be accessible through HTTP
 ✓ The website should be accessible through HTTPS
 ✓ The certificate should not be the default one
 ✓ The Wordpress install page should be visible under http://192.0.2.50/wordpress/
 ✓ MariaDB should not have a test database
 ✓ MariaDB should not have anonymous users

15 tests, 0 failures
```

## Notes

- Either the assignment has the wrong ip `192.168.15.10` stated or I'm going to have to figure that part out as well (check with teacher) <br> Solution:
  - This ip is indeed wrong


## Resources
- [Guide on how to create certificate](https://wiki.centos.org/HowTos/Https)
- [Roles httpd, MariaDB and WordPress from bertvv](https://galaxy.ansible.com/bertvv/)
