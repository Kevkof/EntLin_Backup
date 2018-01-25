# Enterprise Linux Lab Report 00

- Student name: `Kevin Verlinde`
- Github repo: <https://github.com/HoGentTIN/elnx-sme-KevinVerlinde.git>

Purpose: Completing the default setup process that'll be used on all servers.

## Test plan

1. On the host system, go to the local working directory of the project repository
2. Execute `vagrant status`
    - There should be one VM, `pu004` with status `not created`. If the VM does exist, destroy it first with `vagrant destroy -f pu004`
3. Execute `vagrant up pu004`
    - The command should run without errors (exit status 0)
4. Log in on the server with `vagrant ssh pu004` and run the acceptance tests. They should succeed

    ```
    [vagrant@pu004 test]$ sudo /vagrant/test/runbats.sh
    Running test /vagrant/test/common.bats
    ✓ EPEL repository should be available
    ✓ Bash-completion should have been installed
    ✓ bind-utils should have been installed
    ✓ Git should have been installed
    ✓ Nano should have been installed
    ✓ Tree should have been installed
    ✓ Vim-enhanced should have been installed
    ✓ Wget should have been installed
    ✓ Admin user bert should exist
    ✓ Custom /etc/motd should be installed

    10 tests, 0 failures
    ```

    Any tests for the LAMP stack may fail, but this is not part of the current assignment.

5. Log off from the server and ssh to the VM as described below. You should **not** get a password prompt.

    ```
    $ ssh kevin@192.0.2.50
    Welcome to pu004..
    enp0s3     : 10.0.2.15         fe80::a00:27ff:fe5c:6428/64
    enp0s8     : 192.0.2.50        fe80::a00:27ff:fecd:aeed/64
    [kevin@pu004 ~]$
    ```

## Procedure/Documentation

#### site.yml

[Link to the file](../ansible/site.yml)

- Added the correct role

#### all.yml

[Link to the file](../ansible/group_vars/all.yml)

- Added the variables for: 
  - SSH
  - required repo and packages
  - making sure the sshd service is running
  - the message of the day
  - the 'admin' user account

#### common.bats

[Link to the file](../test/common.bats)

- Edited the `admin_user`

## Test report

[Evidence footage](https://youtu.be/3haoNerVFwY)

1. `vagrant status` shows no VM's running.
2. `vagrant up pu004` start the boot process.  
3. `vagrant ssh pu004` should allow ssh into the vm, but doesn't on my system (known issue, without fix. Only have workarounds).
4. Due to `vagrant ssh pu004` not working I went ahead and did the no. 5 of the testplan here: `ssh kevin@192.0.2.50` logs in without prompting for a password.
5. Running the acceptance tests with `sudo /vagrant/test/runbats.sh`

Test results: 
```
Running test /vagrant/test/common.bats
 ✓ EPEL repository should be available
 ✓ Bash-completion should have been installed
 ✓ bind-utils should have been installed
 ✓ Git should have been installed
 ✓ Nano should have been installed
 ✓ Tree should have been installed
 ✓ Vim-enhanced should have been installed
 ✓ Wget should have been installed
 ✓ Admin user kevin should exist
 ✓ Custom /etc/motd should have been installed

10 tests, 0 failures
```

## Resources

- [Ansible Galaxy: rh-base](https://galaxy.ansible.com/bertvv/rh-base/)
- [Passsword Hashing](https://www.mkpasswd.net/index.php?category=crypt)
- [Test file for rh-base](https://github.com/bertvv/ansible-role-rh-base/blob/tests/test.yml)
