# Introduction

As a system administrator, you will need to know how to check the status of a running service, and how to stop, start and restart running services. You'll also need to know how to configure services and fix problems you encounter while running them.

In this lab, you'll look at the list of services running on a Linux machine. You will practice stopping and starting some and query their status.

You'll also fix a problem in a service that is failing to start, and edit the configuration of another service.

## Linux commands reminder

In this lab, we'll use a number of Linux commands that were already explained during Course 3. Here is a reminder of what these commands do:

- ``sudo <command>``: executes a command with administrator rights

-  ``ls <directory>``: lists the files in a directory
- ``mv <old_name> <new_name>``: moves or renames a file from the old name to the new name

- ``tail <file>``: shows the last lines of a file

- ``cat <file>``: prints the whole contents of a file
- ``grep <pattern> <file>``: filters the text of a file according to the pattern
- ``less <file>``: lets you browse a file

Additionally, you can combine these commands using the ``|`` sign. For example:


```
sudo cat /var/log/syslog | grep error | tail
```

will first print the output of ``/var/log/syslog``, then keep only the lines that say "error" and then the last 10 lines of that output.

We will also present a number of new commands such as ``service, logger``. We will briefly explain what these commands do when they are shown. Remember that you can always read the manual page ``using man <command_name>`` to learn more about a command.

While you can copy and paste the commands that are presented in this lab, we recommend typing them out manually, to help with understanding and remembering them.

## Listing system services

Let's look at the services that are installed in the machine. In order to do this, we will use the ``service`` command.

If you run ``service`` with parameters ``--status-all``, it lists the state of services controlled by ``System V``.

If you are interested in seeing only the services that are running, you can use the following command:

```
sudo service --status-all
```

OUTPUT:

```
 [ - ]  avahi-daemon
 [ - ]  cron
 [ - ]  cups
 [ - ]  cups-browsed
 [ - ]  dbus
 [ - ]  exim4
 [ ? ]  hwclock.sh
 [ - ]  procps
 [ + ]  rsyslog
 [ - ]  saned
 [ + ]  ssh
 [ - ]  sudo
 [ + ]  udev
 ```

Here you will see the following notations with respect to the services.

- ``+``: Service is active/running

- ``-``: Service is inactive/stopped

- ``?``: Can't determine whether service is active or not

## Stopping and starting services

Alright, now that we've listed the services let's practice stopping and starting some of them. The first service that we are going to stop is the ``rsyslog`` service. This service is in charge of writing content to the log files, as in ``/var/log/syslog``, ``/var/log/kern.log``, ``/var/log/auth.log`` and others. Processes that generate output will send that output to the ``rsyslog`` service and the service will write it to the corresponding log files depending on how the system was configured.

Let's first start by checking the status of the service. We do this by using the ``service`` command with the ``status`` action:

```
sudo service rsyslog status
```

OUTPUT:

```
rsyslogd is running.
```

This is showing us a lot of information about the service: it's loaded (which means that the OS has the information about the service in memory), it's enabled (which means that it will start automatically on boot), it's active and running. It also tells us where to find some documentation about the service and more. Finally, it shows us the last log lines that this service generated.

We can see this service in action by using the ``logger`` command:

```
logger This is a test log entry
```

The logger command will send the text to the rsyslog service and the service will then write it into ``/var/log/syslog``. We can check that this is the case by looking at the last lines in ``/var/log/syslog``.

```
sudo tail -1 /var/log/syslog
```
OUTPUT: 

```
Jun 18 08:48:21 93431e170ddb student: This is a test log entry
```

Let's now go ahead and stop the rsyslog service:

```
sudo service rsyslog stop
```

We need to execute the command with sudo, because while all users can query the status of services, only users with administrator rights can stop or start services.

OUTPUT:

```
Stopping enhanced syslogd: rsyslogd.
```

To see the current state, we can query the status of the service again:

```
sudo service rsyslog status
```

OUTPUT: 

```
rsyslogd is not running ... failed!
```

We see that the service is now stopped. We can also see what the command logged to ``/var/log/syslog`` when finishing:


```
sudo tail -5 /var/log/syslog
```

OUTPUT: 

```
Jun 18 08:30:02 93431e170ddb rsyslogd: imklog: cannot open kernel log (/proc/kmsg): Operation not permitted.
Jun 18 08:30:02 93431e170ddb rsyslogd: activation of module imklog failed [v8.1901.0 try https://www.rsyslog.com/e/2145 ]
Jun 18 08:30:02 93431e170ddb rsyslogd:  [origin software="rsyslogd" swVersion="8.1901.0" x-pid="984" x-info="https://www.rsyslog.com"] start
Jun 18 08:32:20 93431e170ddb example: This is an example log line
Jun 18 08:48:21 93431e170ddb student: This is a test log entry
```

In the last line, we see that the rsyslog service has exited and is no longer running.

We can try sending text with our logger command again:

```
logger This is another test log entry
```

And then check that the contents of ```/var/log/syslog```:


```
sudo tail /var/log/syslog
```

OUTPUT:
```
Jun 18 08:30:02 93431e170ddb rsyslogd: imklog: cannot open kernel log (/proc/kmsg): Operation not permitted.
Jun 18 08:30:02 93431e170ddb rsyslogd: activation of module imklog failed [v8.1901.0 try https://www.rsyslog.com/e/2145 ]
Jun 18 08:30:02 93431e170ddb rsyslogd:  [origin software="rsyslogd" swVersion="8.1901.0" x-pid="984" x-info="https://www.rsyslog.com"] start
Jun 18 08:32:20 93431e170ddb example: This is an example log line
Jun 18 08:48:21 93431e170ddb studen
```

We can see that nothing was logged, because rsyslog wasn't running. Let's start it back up:

```
sudo service rsyslog start
```

Output:

```
Starting enhanced syslogd: rsyslogdrsyslogd: imklog: cannot open kernel log (/proc/kmsg): Operation not permitted.
rsyslogd: activation of module imklog failed [v8.1901.0 try https://www.rsyslog.com/e/2145 ]
.
```

```
sudo service rsyslog status
```
OUTPUT:
```
rsyslogd is running.
```

And see that it's running again. Let's try our logger command one more time:

```
logger This is another test log entry
```

And then check that the contents of /var/log/syslog:

```
sudo tail /var/log/syslog
```

OUTPUT:

```
Jun 18 08:30:02 93431e170ddb rsyslogd: imklog: cannot open kernel log (/proc/kmsg): Operation not permitted.
Jun 18 08:30:02 93431e170ddb rsyslogd: activation of module imklog failed [v8.1901.0 try https://www.rsyslog.com/e/2145 ]
Jun 18 08:30:02 93431e170ddb rsyslogd:  [origin software="rsyslogd" swVersion="8.1901.0" x-pid="984" x-info="https://www.rsyslog.com"] start
Jun 18 08:32:20 93431e170ddb example: This is an example log line
Jun 18 08:48:21 93431e170ddb student: This is a test log entry
Jun 18 08:52:38 93431e170ddb rsyslogd: imklog: cannot open kernel log (/proc/kmsg): Operation not permitted.
Jun 18 08:52:38 93431e170ddb rsyslogd: activation of module imklog failed [v8.1901.0 try https://www.rsyslog.com/e/2145 ]
Jun 18 08:52:38 93431e170ddb rsyslogd:  [origin software="rsyslogd" swVersion="8.1901.0" x-pid="3326" x-info="https://www.rsyslog.com"] start
Jun 18 08:53:03 93431e170ddbc student: This is another test log entry
```

## Fixing a failing service

In order to list the state of services controlled by ``System V``, you can use the following command:

```
sudo service --status-all
```

OUTPUT: 

```
 [ - ]  avahi-daemon
 [ - ]  cron
 [ - ]  cups
 [ - ]  cups-browsed
 [ - ]  dbus
 [ - ]  exim4
 [ ? ]  hwclock.sh
 [ - ]  procps
 [ + ]  rsyslog
 [ - ]  saned
 [ + ]  ssh
 [ - ]  sudo
 [ + ]  udev
 ```

 Here you will find ``-`` with the ``cups`` service, which means it is inactive/stopped. This is the service used to manage printers on Linux systems. We can get more information about this service by checking the status:


 ```
 sudo service cups status
 ```

 Output:

```
cupsd is not running ... failed!
```

We see here that the ``cups`` service is in a failed state. So, let's look at the contents of that directory:

```
sudo ls -l /etc/cups
```

OUTPUT:
```
total 64
-rw-r--r-- 1 root root 27303 May 19  2023 cups-browsed.conf
-rw-r--r-- 1 root root  2923 Jun 11 20:16 cups-files.conf
-rw-r--r-- 1 root root  6496 Jun 18 08:32 cupsd.conf.old
drwxr-xr-x 2 root root  4096 Jun 11 20:16 interfaces
drwxr-xr-x 2 root root  4096 Jun 11 20:16 ppd
-rw-r--r-- 1 root root   240 Jun 18 08:31 raw.convs
-rw-r--r-- 1 root root   211 Jun 18 08:31 raw.types
-rw-r--r-- 1 root root   142 Jun 11 20:16 snmp.conf
drwxr-xr-x 2 root root  4096 Jun 11 20:16 ssl
```

There's no ``cupsd.conf``, but there is ``cupsd.conf.old``. Apparently the configuration file was deleted. Good thing we kept a copy! Let's move that file so that cups can find it and start successfully:

```
sudo mv /etc/cups/cupsd.conf.old /etc/cups/cupsd.conf
```

As with the other commands, we get no output after executing this. We can run ls again to see that the file was renamed correctly:


```
sudo ls -l /etc/cups
```

OUTPUT:

```
total 64
-rw-r--r-- 1 root root 27303 May 19  2023 cups-browsed.conf
-rw-r--r-- 1 root root  2923 Jun 11 20:16 cups-files.conf
-rw-r--r-- 1 root root  6496 Jun 18 08:32 cupsd.conf
drwxr-xr-x 2 root root  4096 Jun 11 20:16 interfaces
drwxr-xr-x 2 root root  4096 Jun 11 20:16 ppd
-rw-r--r-- 1 root root   240 Jun 18 08:31 raw.convs
-rw-r--r-- 1 root root   211 Jun 18 08:31 raw.types
-rw-r--r-- 1 root root   142 Jun 11 20:16 snmp.conf
drwxr-xr-x 2 root root  4096 Jun 11 20:16 ssl
```

Now that the file was renamed successfully, we can start cups:

```
sudo service cups start
```

And then check the status:


```
sudo service cups status
```

OUTPUT:

```
cupsd is running.
```
We've fixed it!

