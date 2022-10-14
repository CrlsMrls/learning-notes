# Scheduled jobs

Scheduled jobs are tasks that run at preconfigured times.

## cron
Cron is a time-based job scheduling daemon. It runs in the background and execute automatically “cron jobs”.

Check the syntax in `/etc/crontab` file.

### Editing cron tasks

- The following line will run `/usr/bin/touch test_passed`, every 15th of each month at 3 am:
`0 3 15 * * /usr/bin/touch test_passed`
- Run at 11:00AM on every Sunday: `0 11 * * 0 /usr/bin/touch weekly`
- Run at 0:00AM every day: `0 0 1 * * /usr/bin/touch monthly`

- `*` = match all possible values (i.e., every hour)
- `,` = match multiple values (i.e., 15,45)
- `-` = range of values (i.e., 2-4)
- `/` = specifies steps (i.e., */4)

To see the crontab for the root user:
```bash
$ sudo crontab -l
```

Alternatively, we can create script files in the following folders:
- hourly => `/etc/cron.hourly/`
- daily => `/etc/cron.daily/`
- monthly => `/etc/cron.monthly/`
- weekly => `/etc/cron.weekly/`

For example, executing hourly the file `shellscript` (do not terminate with `.sh`):
```bash
$ sudo cp shellscript /etc/cron.hourly/
$ sudo chmod +rx /etc/cron.hourly/shellscript
```

### Verify completion

Check `/var/log/cron` or `/var/log/messages` log files

This will list all commands:
```bash
$ grep CMD /var/log/cron
```
## anacrontab

One of cron's biggest weaknesses is that it assumes that your server or computer is always on. Anacron uses time-stamped files to find out when the last time its commands were executed. 

Check the syntax in `/etc/anacrontab` file.

## Adding and editing anacontab tasks

For example for adding a task with the following specs:
- It should run once every 10 days
- It should have 5 minutes of delay
- The job id should be `db_cleanup`
- The command to run is: `/usr/bin/touch /root/anacron_created_this`

Edit the crontab table:
```bash
$ sudo vim /etc/anacrontab

10 5 db_cleanup /usr/bin/touch /root/anacron_created_file
```

### Verify completion

Result of `anacron` jobs are also in `/var/log/cron` or `/var/log/messages` log files

This will list all anacron output:
```bash
$ grep anacron /var/log/cron
```

To use the `journalctl`, pipe the result to `systemd-cat`
```bash
$ sudo vim /etc/anacrontab
@weekly 45 my_job_name /bin/echo 'Testing anacron' | systemd-cat --identifier=test_job

$ journalctl | grep test_job
```

### Force to execute crontab

Crontab will all pending tasks for today right now, sue the `now` flag:

```bash
$ sudo anacron -n
```

If the anacron was executed for today, re-run the tasks, use the `force` flag:
```bash
$ sudo anacron -n -f
```

## at 

Runs only once a program for later execution  "at" sometime.

### execute commands at a specified time

```bash
$ at '15:30 August 20 2024'
warning: commands will be executed using /bin/sh
at> /usr/bin/touch at_file_name_scheduler
(Ctrl + D)
job 2 at Tue Aug 20 15:30:00 2024
```

All these are valid options:

```bash
$ at 15:00
$ at 'August 20 2022'
$ at '2:30 August 20 2022'
$ at 'now + 30 minutes'
$ at 'now + 3 hours'
$ at 'now + 3 days'
$ at 'now + 3 weeks'
$ at 'now + 3 months'
```

### List the user's pending jobs

The command to see the jobs that are scheduled to run in at utility run `atq`.

Following the previous example:
```bash
$ atq
2       Tue Aug 20 15:30:00 2024 a bob
```

To check what is the command associated run `atq -c <job-number>`

### Delete jobs

Remove all at jobs that exist is done with `atrm`, but first check the job number with `atq`.

```bash
$ atq
1       Sun Aug 20 15:30:00 2034 a bob
$ atrm 1
$ atq
$
```

#### Verify completion

```bash
at 'now + 1 minute' 
at > echo "my at job' 
(Ctrl + D)
```

To check output:

```bash
sudo grep atd /var/log/cron
```

To use the `journalctl`, pipe the result to `systemd-cat`
```bash
$ at 'now + 1 minute' 
at > echo "my at job' | systemd-cat --identifier=my_at_task_name
(Ctrl + D)

$ journalctl | grep my_at_task_name
```
