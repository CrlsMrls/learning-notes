# Systemd Linux Service Manager

`systemd` is a system and service manager for Linux operating systems. Started during early boot, it runs as PID 1 and starts the rest of the system. It is used to boot up the machine, manage services, auto mount file systems, log events, setup hostname, and other system tasks.

The `systemctl` command is `systemd`'s primary command-line interface and it is the central management tool for controlling the init system. 

The `journalctl` command is used to query the `systemd` journal.

## Boot process

The Linux boot and startup process has the following steps:
- BIOS POST
- Boot loader (GRUB2)
- Kernel initialization
- Start systemd, the parent of all processes.

The first step checks the hardware and then it issues a BIOS interrupt, INT 13H. It loads the boot sector into the RAM and control is then transferred to that code. There are three boot loaders used by most Linux distributions, GRUB, GRUB2, and LILO. GRUB2 is the most common and newest of the options.

The primary function of the boot loader is to get the Linux kernel loaded into memory and running.

For a system booting through `legacy BIOS` mode, the grub configuration file is in `/etc/default/grub`. Among other settings, this allows to pass parameters to the kernel.

After modifying this file, run the following command to generate the new grub configuration file: `sudo grub2-mkconfig -o /boot/grub2/grub.cfg`

To install grub in a disk run `sudo grub2-install /dev/sda`. To list the available disks use the `lsblk` command.

Once the kernel has decompressed itself, initializes the heap and the hardware (among many others), it loads `systemd`.

## Systemd units

Systemd categorizes resources in '_units_' based on their type. These can be services, sockets, mount points, devices, etc. A systemd unit consists of a name, type, and configuration file. The name provides a unique identity to the unit. 

- `.service`: A service unit describes how to manage a daemon: how to start/stop, the dependencies and ordering.
- `.socket`: A socket unit file describes a network or IPC socket. 
- `.device`: A unit that describes a filesystem. 
- `.mount`: This unit defines a mountpoint on the system to be managed by systemd.
- `.swap`: This unit describes swap space on the system. 
- `.target`: A target unit is used to provide synchronization points for other units when booting up or changing states.
- other unit types are `.path`, `.timer`, `.snapshot`, `.slice`, `.scope`, etc.

### Display all units state

To display the **status of all unit types at startup** use `systemctl list-unit-files`. 

```bash
$ systemctl list-unit-files
UNIT FILE                              STATE   
proc-sys-fs-binfmt_misc.automount      static  
sys-fs-fuse-connections.mount          masked  
sys-kernel-config.mount                static  
tmp.mount                              disabled
sshd-keygen@.service                   disabled
sshd.service                           enabled 
sshd.socket                            enabled
sshd@.service                          static  
syslog.service                         bad     
system-update-cleanup.service          static 
```
From these, the most common states are:
- the `enabled` state: the systemd starts it at boot time.
- the `disabled` state: the systemd does not start it at startup.
- the `static` state: neither the systemd starts it at startup nor allows us to change its state.

### Unit configuration files
New `systemd` unit installations store its default configuration at `/lib/systemd/system`. Do not edit these files.

To modify the unit configurations, go to `/etc/systemd/system` directory. For example, a unit called `example.service`, will be in a subdirectory called `example.service.d` and within this directory the file ending with `.conf` configures the system's unit. 

The run-time unit definitions are at `/run/systemd/system`. 

## Systemd Targets

Targets are synchronization points that `systemd` uses to bring the server into a specific state. 

A *target* of systemd is used for grouping units (e.g. services) and synchronization points during start-up. The most common targets:

- `multi-user.target` -> non-graphical multi-user system.
- `graphical.target` -> Set up a graphical multi-user system.
- `rescue.target` -> Set up a rescue shell. It pulls in the base system and spawns a rescue shell.
- `emergency.target` -> unit target that starts an emergency shell on the main console. Emergency mode provides the most minimal environment possible.

### Units in a target

To see what units are tied to a target, you can type:

```bash
$ systemctl list-dependencies multi-user.target
```

### Current target

To list the currently loaded target unit:

```bash
$ systemctl get-default
graphical.target
```

### Change default target

To change the default target that will be used at boot:

```bash
$ sudo systemctl set-default graphical.target
Removed /etc/systemd/system/default.target.
Created symlink /etc/systemd/system/default.target → /usr/lib/systemd/system/graphical.target.
```

### Execute target 

To run the target system immediately use `systemctl isolate`.

```bash
$ sudo systemctl isolate rescue.target

Broadcast message from root@localhost on pts/0:
The system is going down to rescue mode NOW!
```

### Stopping or Rebooting 

`systemd` shutdown is another target. It will unmount file systems, kill processes and release any remaining resources, in a controlled fashion.

For shuting down or rebooting a system use the `shutdown` command. In the following example powers off in two hours:

```bash
$ sudo shutdown +120
Shutdown scheduled for Fri 2022-10-14 17:11:27 UTC, use 'shutdown -c' to cancel.
```

The `shutdown` command allows to reboot with the `-r` flag.

```bash
$ sudo shutdown -r 02:00 'Scheduled restart at 2am for x reason'
```

`shutdown` is a symbolic link to `/bin/systemctl`. This is to be backwards compatible.

```bash
$ sudo systemctl reboot
$ sudo systemctl poweroff
```

To force the reboot, in case some processes do not stop, use the `--force` flag. In case that does not work, use the flag twice: `--force --force` 

## Unit Commands

The following commands focus on service units, but apply for all `systemd` units. 


### Overview of the system state

To list units that `systemd` currently has in memory. 
```bash
$ systemctl list-units 
```

To list the units that `systemd` loaded or attempted to load into memory add the `--all` flag:

```bash
$ systemctl list-units --all
```

The units that are filtered by `--type=` and `--state=`.

To list all loaded services on your system:

```bash
$ systemctl list-units --type=service
```

And to list all active services:
```bash
$ systemctl list-units --type=service --state=active
```

### List dependencies

Some services are dependent on others. To list the dependencies:
```bash
$ sudo systemctl list-dependencies sshd.service
sshd.service
  ├─system.slice
  ├─sshd-keygen.target
  │ ├─sshd-keygen@ecdsa.service
  │ ├─sshd-keygen@ed25519.service
  │ └─sshd-keygen@rsa.service
  └─sysinit.target
    ├─dev-hugepages.mount
...
```

Note that this command only lists units currently loaded into memory by the service manager.

### Check the status

The following command shows the ssh daemon is `enabled` and PID is `920`.

```bash
$ systemctl status sshd.service
  sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2022-10-14 13:19:29 UTC; 57min ago
     Docs: man:sshd(8)
           man:sshd_config(5)
 Main PID: 920 (sshd)
    Tasks: 1 (limit: 1340704)
   Memory: 16.9M
   CGroup: /system.slice/sshd.service
           └─920 /usr/sbin/sshd -D -oCiphers=aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes25>
```

### Edit the configuration

To make a modification to a unit file:

```bash
$ sudo systemctl edit --full sshd.service
```

### Reload the configuration

Reload the systemd manager configuration. This will reload all unit files and recreate the entire dependency tree. 

After modifying a unit file, reload the `systemd` process itself to pick up the changes:

```bash
$ sudo systemctl daemon-reload
```

### Start and stop services

For clarity, use the `.service` suffix to be explicit about the target.

```bash
$ sudo systemctl start application.service
$ sudo systemctl stop application.service
```

Some services allow to `reload` the configuration (without restarting). If the new configuration fails, it keeps running it with the previous config. If unsure whether the service has the functionality to reload its configuration, use `reload-or-restart`:

```bash
$ sudo systemctl reload-or-restart application.service
```

### Enable and disable services

`Enabled` means the system will run the service on the next boot.

```bash
$ sudo systemctl is-enabled sshd.service
enabled
$ sudo systemctl disable sshd.service
Removed /etc/systemd/system/multi-user.target.wants/sshd.service.
$ sudo systemctl is-enabled sshd.service
disabled
$ sudo systemctl enable sshd.service
Created symlink /etc/systemd/system/multi-user.target.wants/sshd.service → /usr/lib/systemd/system/sshd.service.
```

If you do not want to restart the server, use `--now` flag:
```bash
$ sudo systemctl enable --now sshd.service
$ sudo systemctl disable --now sshd.service
```

### Completely disable 

To completely disable a service (so that any start operation on it fails) use `mask`. This is a stronger version of `disable`, since it prohibits all kinds of activation.

A disabled service may still run because a dependency. To disable it permanently mask it:

```bash
$ sudo systemctl mask httpd.service
Created symlink /etc/systemd/system/httpd.service → /dev/null.
$ sudo systemctl enable --now httpd.service
Failed to enable unit: Unit file /etc/systemd/system/httpd.service is masked.
```

To unmask a unit, making it available for use again, use `unmask`:

```bash
$ sudo systemctl unmask httpd.service
Removed /etc/systemd/system/httpd.service.
$ sudo systemctl enable --now httpd.service
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```

## Getting information

The `journalctl` searches the contents of the `systemd` journal, the logs for entries related to a unit.

By default Linux logs all applications and the kernel into `/var/log` directory. Alternatively to `journalctl`, use `grep` in that directory, e.g. to search for all files containing the `reboot` string run `sudo grep -r 'reboot' /var/log/`.


### Logs from a unit

To show messages for the specified systemd unit use `--unit` flag.

For example, to find out what IP address last connected to this daemon successfully:

```bash
$ sudo journalctl --unit=sshd.service

-- Logs begin at Fri 2022-10-14 13:19:28 UTC, end at Fri 2022-10-14 14:24:19 UTC. --
Oct 14 13:19:29 centos-host sshd[920]: Server listening on 0.0.0.0 port 22.
Oct 14 13:19:29 centos-host sshd[920]: Server listening on :: port 22.
Oct 14 14:08:48 centos-host sshd[1274]: Connection closed by authenticating user root 10.34.124.10 port>
Oct 14 14:08:48 centos-host sshd[1296]: Accepted password for root from 10.34.124.10 port 40844 ssh2
```

Similarly, you can use the process id, the following example uses the PID 920:

```bash
$ sudo journalctl _PID=920
-- Logs begin at Fri 2022-10-14 13:19:28 UTC, end at Fri 2022-10-14 14:37:48 UTC. --
Oct 14 13:19:29 centos-host sshd[920]: Server listening on 0.0.0.0 port 22.
Oct 14 13:19:29 centos-host sshd[920]: Server listening on :: port 22.
```

### Logs from a program

To check logs for a specific program: `journalctl /bin/sudo`

### Filter results by error log level

To filter error level messages `journalctl -p err`

The different levels are the usual syslog log levels: `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info` and `debug`. When a single log level is specified, all messages with this log level or a lower (hence more important) log level are shown.

### Filter results with regexp

To filter using regular expression use `-g` or `--grep`

The matching is case sensitive. This can be overridden with the `--case-sensitive` option.

For example, to filter `info` priority logs that begin with letter `c` use `journalctl -p info -g '^c'`

## References

For more details:
- https://www.freedesktop.org/software/systemd/man/systemd.html
- https://www.freedesktop.org/software/systemd/man/systemctl.html
- https://www.freedesktop.org/software/systemd/man/journalctl.html 