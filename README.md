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
  --onsuccess \
      'mail foo@bar.org -s "Connection to $1:$2 succeeded!" <<< "$(traceroute -m 1 $1)"' \
  --onfailure \
      'mail foo@bar.org -s "Connection to $1:$2 failed!" <<< "$(traceroute $1)"'
```

```
# This will check every 60 seconds if /mnt/backup is mounted.
# A notification is displayed every 2 minutes if not mounted,
# nothing will be displayed if already mounted.
check-and-monitor \
   --command 'mountpoint $1 2>&1' \
   --onsuccess '' \
   --onfailure 'notify-send "$1 is NOT mounted!"' \
   --monitor-failure 120 \
   /mnt/backup
```

## license
    Copyright (C) 2021-2025 luc.deschenaux@freesurf.ch

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU Affero General Public License as published
    by the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU Affero General Public License for more details.

    You should have received a copy of the GNU Affero General Public License

## how to
```
NAME
       check-and-monitor - monitor connection to tcp server or anything else

SYNOPSIS
       check-and-monitor [OPTION]... [<arg> ...]

DESCRIPTION
       Unless you specify something else to check (using the --command option
       below), this command will check tcp connection to <arg1> on port <arg2>
       every <--interval> seconds, run <--onfailure> command after <--failures>
       attempts, and forces re-notification after <--monitor> seconds.
       All the options below are then optional.

       -h, --help

       -l, --load-config <filename>
               Load configuration from file
               Options specified after -l will override loaded settings

       -i, --interval <60>
               Sleep interval between attempts

       -t, --timeout  <10>
               Attempt timeout in seconds

       -f, --failures <3>
               Notify failure after specified failed attempts

       -s, --successes <0>
               Notify success after specified succeeded attempts

       -n, --notify-command <notify-send>
               Default command for default --onsuccess and --onfailure
               Variables of interest:
                 $1                   - The "--onsuccess" or
                                         "--onfailure" message, evaluated
                 $command_exit_code   - The exit code of --command
                 $argv[]              - Command line parameters
                                        (see example configuration file below)

       -Y, --onsuccess <'$notify "Connection to $1:$2 succeeded!"'>
               Command for notifying successful attempts.
               Can be set to empty string.

       -N, --onfailure <'$notify "Connection to $1:$2 failed!"'>
               Command for notifying failed attempts.
               Can be set to empty string.

       -v, --verbose
               Report command output not only on status change

       -m, --monitor <86400>
                Seconds before success or failure re-notification
                -1: disable re-notifications

       -S, --monitor-success <86400>
                Seconds before success re-notification (overrides --monitor)
                -1: disable re-notifications

       -F, --monitor-failure <3600>
                Seconds before failure re-notification (overrides --monitor)
                -1: disable re-notifications

       -c, --command <'nc -w $timeout -v -z $1 $2 2>&1'>
                Command to run for checking whatever the check.
                When you change this for something where $1 and $2 means
                anything else than host:port you need to set -Y and -N.

       -o, --output <cat>
                Allow redirecting stdout and stderr
                eg:
                    -o logger
                or
                    -o 'ts "[%Y-%m-%d %H:%M:%S]"'

       -p, --pid-file <filename>
                Do not run the script if another process with matching command
                line is running with the same pid, else store the pid there


CONFIGURATION FILE
       When file ~/.check-and-monitor.conf exists it is evaluated before
       command line options.
       Command line options override settings from ~/.check-and-monitor.conf.

       You can also specify a configuration file with -l|--load-config.
       Options specified after -l will override settings from the config file.

       DEFAULT CONFIGURATION:
           timeout=10
           interval=60
           failures_threshold=3
           successes_threshold=0
           monitor_success=86400
           monitor_failure=900
           notify=echo
           onsuccess='\$notify "Connection to \$1:\$2 succeeded!"'
           onfailure='\$notify "Connection to \$1:\$2 failed!"'
           command='nc -w \$timeout -v -z \$1 \$2 2>&1'
           output=cat

       EXAMPLE CONFIGURATION FILE (examples/mail-send.conf)
           # Send a mail to user@host on host un/reachable
           # with full traceroute output on failure,
           # and only the first gateway on success
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
           monitor_failure=900
           output=logger

       EXAMPLE CONFIGURATION FILE (examples/wg-check.conf)
           # Restart wireguard tunnel after 2 failed pings through the tunnel.
           # Use eg in cron with:
           #  check-and-monitor -p /var/run/wgcheck-wg0.pid -l examples/wgcheck.conf wg0 10.0.42.1

           wg_restart() {
           IFACE=${argv[0]}
             ip link show $IFACE > /dev/null 2>&1 && wg-quick down $IFACE
             wg-quick up $IFACE
           }

           timeout=5
           interval=25
           failures_threshold=2
           monitor_success=-1
           monitor_failure=0
           onsuccess='echo "${argv[0]}: host ${argv[1]} is up!"'
           onfailure=wg_restart
           command='ping -c 1 -w $timeout ${argv[1]} 2>&1'
           output=logger


```
