# Cheat sheets and checklists

- Student name: NAME
- Github repo: URL

## Basic commands

| Task                | Command |
| :---                | :---    |
| Query IP-adress(es) | `ip a`  |

## Git workflow

Simple workflow for a personal project without other contributors:

| Task                                         | Command                   |
| :---                                         | :---                      |
| Current project status                       | `git status`              |
| Select files to be committed                 | `git add FILE...`         |
| Commit changes to local repository           | `git commit -m 'MESSAGE'` |
| Push local changes to remote repository      | `git push`                |
| Pull changes from remote repository to local | `git pull`                |

## Checklist network configuration

1. Is the IP-adress correct? `ip a` / `ifconfig`
2. Is the router/default gateway correct? `ip r`
3. Is a DNS-server available? `cat /etc/resolv.conf`

## VyOS
| Task                        | Command                                               |
| :---                        | :---                                                  |
| Keyboard layout             | `sudo dpkg-reconfigure keyboard-configuration`        |
| Set IP address on interface | `set interfaces ethernet eth0 address 192.168.0.1/24` |

```
vyos@vyos $ configure
vyos@vyos # [configuration commands]
vyos@vyos # commit
vyos@vyos # save
vyos@vyos # exit
vyos@vyos $
```
