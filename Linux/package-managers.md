# Linux Package Managers 

- [Update packages](#update-packages)
- [Search packages](#search-packages)
- [Install and remove packages](#install-and-remove-packages)
- [Repositories](#repositories)
- [Checking history](#checking-history)
- [Install group of packages](#install-group-of-packages)
- [Identify Files](#identify-files)

A package manager installs, manages, and uninstalls applications. 

Two of the most common package managers are `apt` and `dnf`. `apt` manages `DEB` packages, while `dnf` manages `RPM` packages. 

## Update packages

Upgrade all applications:

### apt
In `apt` based distros, updating packages requires two steps:

```bash
$ sudo apt update
$ sudo apt upgrade
```

`apt` caches package information that requires to update first before upgrading the packages.

### dnf
In `dnf`, it is just one command:

```bash
$ sudo dnf upgrade
```

`dnf update` is equal to `dnf upgrade`

Use `check-upgrade` to list the old packages that can be upgraded:

```bash
$ sudo dnf check-upgrade
```

## Search packages

### apt
```bash
$ sudo apt search <query-string>
$ sudo apt show <package-name>
```

### dnf
```bash
$ sudo search <query-string>
$ sudo dnf info <package-name>
```
## Install and remove packages

### apt
```bash
$ sudo apt install <package-name>
```

Run `purge` to remove configuration files, in addition to app data:

```bash
$ sudo apt remove <package-name>
$ sudo apt purge <package-name>
```

Use `autoremove` to remove no longer needed dependencies:
```bash
$ sudo apt autoremove
```

### dnf
`dnf` allows to install a package from downloaded file:

```bash
$ sudo dnf install <package-name>
$ sudo dnf reinstall <package-name>
$ sudo dnf install /path/to/package.rpm
```

As an example, this installs `nginx`:
```bash
$ sudo dnf install nginx
Last metadata expiration check: 0:00:49 ago on Mon Oct 17 13:28:46 2022.
Dependencies resolved.
========================================================================================================
 Package                        Arch      Version                                    Repository    Size
========================================================================================================
Installing:
 nginx                          x86_64    1:1.14.1-9.module_el8.0.0+1060+3ab382d3    appstream    570 k
Installing dependencies:
 nginx-all-modules              noarch    1:1.14.1-9.module_el8.0.0+1060+3ab382d3    appstream     23 k
 nginx-filesystem               noarch    1:1.14.1-9.module_el8.0.0+1060+3ab382d3    appstream     24 k
 ...
```

Use `remove` to remove the packages:
```bash
$ sudo dnf remove <package-name>
$ sudo dnf remove /path/to/package.rpm
```

Run `autoremove` to remove no longer needed dependencies:
```bash
$ sudo dnf autoremove
```

## Repositories

### dnf
List the optional enabled/disabled repositories:
```bash
$ sudo dnf repolist -all
repo id                               repo name                                                 status
appstream                             CentOS Stream 8 - AppStream                               enabled
appstream-source                      CentOS Stream 8 - AppStream - Source                      disabled
baseos                                CentOS Stream 8 - BaseOS                                  enabled
baseos-source                         CentOS Stream 8 - BaseOS - Source                         disabled
debuginfo                             CentOS Stream 8 - Debuginfo                               disabled
extras                                CentOS Stream 8 - Extras                                  enabled
...
```

Use `-v` flag to list the repositories in detailed format:
```bash
$ sudo dnf repolist -v
  Repo-id : docker-ce-stable
  Repo-name : Docker CE Stable - x86_64
  Repo-revision : 1637198749
  Repo-updated : Wed 17 Nov 2021 07:25:49 PM CST
  Repo-pkgs : 56
  Repo-available-pkgs: 56
  Repo-size : 1.3 G
  Repo-baseurl : https://download.docker.com/linux/centos/8/x86_64/stable
  Repo-expire : 172,800 second(s) (last: Sun 21 Nov 2021 07:37:29 PM CST)
  Repo-filename : /etc/yum.repos.d/docker-ce.repo
```

Use the `config-manager` to enable and disable an optional repository. E.g. enable and disable `powertools` repo:
```bash
$ sudo dnf config-manager --enable powertools
$ sudo dnf config-manager --disable powertools
```

Add a 3rd party repository:
```bash
$ sudo dnf config-manager --add-repo <URL>
```

To remove the 3rd party repo, check the `repo-filename` field and just delete the file:
```bash
$ sudo rm /etc/yum.repos.d/docker-ce.repo
```

## Checking history

apt stores recent history under `/var/log/apt/` folder, check for the `/var/log/apt/history.log` file.

dnf offers a `history` command:

```bash
$ sudo dnf history
```

## Install group of packages

Groups are virtual collections of packages.

```bash
$ dnf group list
Available Environment Groups:
   Server with GUI
   Server
   Minimal Install
   Workstation
   Virtualization Host
   Custom Operating System
Installed Groups:
   Development Tools
Available Groups:
   Container Management
   .NET Core Development
   RPM Development Tools
   Graphical Administration Tools
   Headless Management
   Legacy UNIX Compatibility
   Network Servers
   Scientific Support
   Security Tools
   Smart Card Support
   System Tools

$ dnf group info 'Container Management'
Group: Container Management
 Description: Tools for managing Linux containers
 Mandatory Packages:
   buildah
   containernetworking-plugins
   podman
 Optional Packages:
   python3-psutil
```

These groups can be installed or removed:

```bash
$ sudo dnf group install <group-name>
$ sudo dnf group remove <group-name>
$ sudo dnf group upgrade <group-name>
```

When removing, dnf removes packages in the group which do not belong to another installed group and were not installed explicitly by the user.


## Identify Files

Use `provides` to find what package (installed or not) installed a specific file:

```bash
$ dnf provides /etc/anacrontab
cronie-anacron-1.5.2-4.el8.x86_64 : Utility for running regular jobs
Repo        : baseos
Matched from:
Filename    : /etc/anacrontab
```

The previous example says `cronie-anacron` is the name of the package that created `/etc/anacrontab` file.


Use `repoquery` to list all files installed by a package:

```bash
$ dnf repoquery --list nginx
/etc/logrotate.d/nginx
/etc/nginx/fastcgi.conf
/etc/nginx/fastcgi.conf.default
...
```