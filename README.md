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
      'mail foo@bar.org -s "Connection to $1:$2 succeeded!" <<< "www is up"' \
  --notify-failure \
      'mail foo@bar.org -s "Connection to $1:$2 failed!" <<< "www is down"'
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

       -i, --interval <60> 
               sleep interval between attempts

       -t, --timeout  <10>
               attempt timeout in seconds

       -f, --failures <3>
               notify failure after specified failed attempts

       -s, --successes <0>
               notify success after specified succeeded attempts

       -T, --notify-success <'notify-send "Connection to $1:$2 successed!"'>
               command for notifying successful attempts

       -G, --notify-failure <'notify-send "Connection to $1:$2 failed!"'>
               command for notifying failed attempts
               if not specified with -T, notify-success will be unset.

       -n, --notify-command <notify-send>
               default command for --notify-success and --notify-failure

       -v, --verbose
               print command output not only on status change

       -m, --monitor <86400>
                seconds before success or failure re-notification
       
       -S, --monitor-success <86400>
                seconds before success re-notification

       -F, --monitor-failure <86400> 
                seconds before failure re-notification

       -c, --command <'nc -w $timeout -v -z $1 $2 2>&1'>
                Command to run for checking whatever the check.
                When you change this for something where $1 and $2 means
                anything else than host:port you need to set -T and -G.
```
