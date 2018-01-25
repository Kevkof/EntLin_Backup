# Enterprise Linux Lab Report 03

- Student name: `Kevin Verlinde`
- Github repo: <https://github.com/HoGentTIN/elnx-sme-KevinVerlinde.git>

- Learning how to work with
  - Samba file server
  - FTP server
  - Vsftpd server
  - SELinux
  - Acces to files and directories

## Test plan

1. On the host system, go to the local working directory of the project repository
2. Execute `vagrant status`
    - The VM, `pr011` should have status `not created`. If the VM does exist, destroy it first with `vagrant destroy -f pr011`
3. Execute `vagrant up pr011`
    - The command should run without errors (exit status 0)
4. Log in on the server with `vagrant ssh pr011` and run the acceptance tests with `sudo /vagrant/tests/runbats.sh`.

- 10 tests for the normal install (those have already been tested and documented in [00-report.md](00-report.md))
- 22 tests for samba
- 17 tests for vsftpd

- The shares shoud be accessible from the host, on windows going to `\\files\it` or on Linux `smb://files/it` and logging in with our admin user should show the folder and not an error
- The users: [stefaanv, evyt, elenaa, anc, benoitp, krisv, alexanderd, svena, leend, stevenv, stevenh] this can be done with `ssh -l <user_name> <ip>`

## Procedure/Documentation

#### vagrant-hosts.yml
[Link to file](../vagrant-hosts.yml)

- Add entry for `pr011` with the correct ip (`172.16.0.11`)

#### site.yml
[Link to file](../ansible/site.yml)

- Add entry for `pr011` with the roles:
  - bertvv.samba
  - bertvv.vsftpd
- Add post_tasks to set acl's for the shares

#### pr011.yml
[Link to file](../ansible/host_vars/pr011.yml)

- Set samba features
  - samba users
  - netbios name
  - samba shares (directory mode is needed for vsftd)
- Add rhbase users entries for all users with
  - disable shell login for the non-it members
- VsFtpd Variables
  - disable anonymous
  - enable local
  - set root
  - banner message

#### all.yml
[Link to file](../ansible/group_vars/all.yml)

- Add admin user to needed groups

## Test report

1. `vagrant status` shows no VM's running.
2. `vagrant up pr011` start the boot process.
3. `vagrant ssh pr011` to log into the vm
4. Running the acceptance tests with `sudo /vagrant/test/runbats.sh`

```
✓ The ’nmblookup’ command should be installed
✓ The ’smbclient’ command should be installed
✓ The Samba service should be running
✓ The Samba service should be enabled at boot
✓ The WinBind service should be running
✓ The WinBind service should be enabled at boot
✓ The SELinux status should be ‘enforcing’
✓ Samba traffic should pass through the firewall
✓ Check existence of users
✓ Checks shell access of users
✓ Samba configuration should be syntactically correct
✓ NetBIOS name resolution should work
✓ read access for share ‘public’
✓ write access for share ‘public’
✓ read access for share ‘management’
✓ write access for share ‘management’
✓ read access for share ‘technical’
✓ write access for share ‘technical’
✓ read access for share ‘sales’
✓ write access for share ‘sales’
✓ read access for share ‘it’
✓ write access for share ‘it’

22 tests, 0 failures
```

```
✓ VSFTPD service should be running
✓ VSFTPD service should be enabled at boot
✓ The ’curl’ command should be installed
✓ The SELinux status should be ‘enforcing’
✓ FTP traffic should pass through the firewall
✓ VSFTPD configuration should be syntactically correct
✓ Anonymous user should not be able to see shares
✓ read access for share ‘public’
✓ write access for share ‘public’
✓ read access for share ‘management’
✓ write access for share ‘management’
✓ read access for share ‘technical’
✓ write access for share ‘technical’
✓ read access for share ‘sales’
✓ write access for share ‘sales’
✓ read access for share ‘it’
✓ write access for share ‘it’

17 tests, 0 failures
```
##### In linux

![Linux](https://i.imgur.com/Gj1q3sa.png)

#### In windows and linux VM

![Windows](https://i.imgur.com/GAJ6Gh5.png)


## Resources

- [Ansible role Samba (by bertvv)](https://galaxy.ansible.com/bertvv/samba/)
- [Ansible role VsFtpd (by bertvv)](https://galaxy.ansible.com/bertvv/vsftpd/)
- Talking with fellow students about the assignment
