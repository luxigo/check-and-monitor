# check-and-monitor

## examples
```
# This will check every 60 seconds (--interval) if a tcp connection can be
# established with foo.org on port 80.
# Check result is sent to stdout only on state change (--verbose not specified)
# Host is considered down after 3 consecutive failed attempts (--failures)
# Host is considered up after 1 successful attempt (--success)
# A mail will be sent on state change (--notify-success and --notify-failure),
# When the host is down a reminder is sent every hour (--monitor-failure)
# When the host is up a reminder is sent every day (--monitor-success)
check-and-monitor foo.org 80 \
  --monitor-failure 3600 \
  --notify-success \
      'mail foo@bar.org -s "Connection to $1:$2 succeeded!" <<< "$(traceroute -m 1 $1)"' \
  --notify-failure \
      'mail foo@bar.org -s "Connection to $1:$2 failed!" <<< "$(traceroute $1)"'
```

```
# This will check every 60 seconds if /mnt/backup is mounted.
# A notification is displayed every 2 minutes if not mounted,
# nothing will be displayed if already mounted.
check-and-monitor \
   --command 'mountpoint $1 2>&1' \
   --notify-success '' \
   --notify-failure 'notify-send "$1 is NOT mounted!"' \
   --monitor-failure 120 \
   /mnt/backup
```

## license
AGPL-3.0 or later

## how to
```
NAME
       check-and-monitor - monitor connection to tcp server or anything

SYNOPSIS
       check-and-monitor [OPTION]... [<server> <port>]

DESCRIPTION
       Unless you specify something else to check (using the --command option
       below), this command will check tcp connection to <server> <port>
       every <--interval> seconds, run notification command after <--failures>
       attempts, and forces re-notification after <--monitor> seconds.
       All the options below are optional.

       -h, --help

       -l, --load-config <filename>
               load configuration from file

       -i, --interval <60>
               sleep interval between attempts

       -t, --timeout  <10>
               attempt timeout in seconds

       -f, --failures <3>
               notify failure after specified failed attempts

       -s, --successes <0>
               notify success after specified succeeded attempts

       -n, --notify-command <notify-send>
               default command for --notify-success and --notify-failure

       -Y, --notify-success <'$notify "Connection to $1:$2 succeeded!"'>
               command for notifying successful attempts

       -N, --notify-failure <'$notify "Connection to $1:$2 failed!"'>
               command for notifying failed attempts

       -v, --verbose
               report command output not only on status change

       -m, --monitor <86400>
                seconds before success or failure re-notification

       -S, --monitor-success <86400>
                seconds before success re-notification

       -F, --monitor-failure <86400>
                seconds before failure re-notification

       -c, --command <'nc -w $timeout -v -z $1 $2 2>&1'>
                Command to run for checking whatever the check.
                When you change this for something where $1 and $2 means
                anything else than host:port you need to set -Y and -N.

CONFIGURATION FILE
       When file ~/.check-and-monitor.conf exists it is evaluated before
       command line options.
       Command line options override settings from ~/..check-and-monitor.conf.

       You can also specify a configuration file with -l|--load-config.
       Options specified after -l will override settings from the config file.

       DEFAULT CONFIGURATION:
           timeout=10
           interval=60
           failures_threshold=3
           successes_threshold=0
           monitor_success=86400
           monitor_failure=86400
           notify=notify-send
           notify_success='$notify "Connection to $1:$2 succeeded!"'
           notify_failure='$notify "Connection to $1:$2 failed!"'
           command='nc -w $timeout -v -z $1 $2 2>&1'

       EXAMPLE CONFIGURATION FILE:
           dest=user@host
           mail-send() {
               if [ $command_exit_code -eq 0 ] ; then
                  msg=$(traceroute -m 1 ${argv[0]})
                else
                  msg=$(traceroute ${argv[0]})
               fi
               mail $dest -s "$1" <<< "msg"
           }
           notify=mail-send

```
