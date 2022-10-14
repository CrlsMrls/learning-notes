
# Processes 

A process in Linux is a program in execution. 

## Display information about active processes

To see every process on the system using standard syntax:
- `ps -e`
- `ps -ef`
- `ps -eF`
- `ps -ely`

To see every process on the system using BSD syntax:
- `ps ax`
- `ps axu`

Using `ps lax` command we can see all processes running on the system, along with their nice values.

To search for a specific process name use `pgrep` which is similar to running a `ps` with `grep`.  

```bash
$ pgrep -a sshd
920 /usr/sbin/sshd -D -oCiphers=aes256-gcm@openssh.com...
```

## Stop a process

The command kill sends a specified signal to the processes. 

These are the signal names and their corresponding numbers:

```bash
$ kill -L
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```


## Process priority

The `nice` is the value of the current process priority.

Processes with a higher priority will be executed before those with a lower priority, while processes with the same priority are scheduled one after the next, repeatedly.

To assign a nice value to the a process use `renice`

```bash
$ sudo renice 9 920
920 (process ID) old priority 0, new priority 9
```

To start a process and give it a nice value other than the default one, use:

```bash
$ nice -n [value] [process name]
```


## List open files

`lsof` lists information about files opened by processes.

The following example lists all opened files by process id 1:
```bash
$ sudo lsof -p 1 
```

## Links

- Linux ps command: https://www.digitalocean.com/community/tutorials/linux-ps-command
