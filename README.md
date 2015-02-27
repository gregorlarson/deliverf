deliverf
========

Utility to support delayed delivery of Postfix mail to a file.

Introduction
=========
Postfix will deliver mail to a file in mbox format. By default, `/var/mail/{username}`
(in my distro). You can also indicate an arbritrary file. There are a number of ways to make it do this. A common example is a line in `/etc/aliases` as follows:
```
bob:  /home/bob/Inbox
```
Another common example is in `~/.forward` like:
```
/mnt/nethome/bob/Mail/Inbox
```

One problem with local mail delivery to a file is that the directory must be mounted
and available at all times when postfix is running, otherwise, postfix will return
an error to the sending agent. Postfix will not re-attempt local delivery if the
mount is missing. Depending on how your mail server is configured, it may even
generate a separate bounce message or [backscatter].

This simple script allows postfix to hold and re-attempt delivery of the message
if the filesystem is not available. It does this by returning an exit code of
75 (`EX_TEMPFAIL`) when it cannot lock and write the destination file.

Usage
=====
A basic example of how this can be used in `/etc/aliases` is:
```
bob:    "|deliverf /mnt/users/bob/Inbox"
```
or
```
bob:    "|deliverf -n /mnt/users/bob/Inbox"
```
or use `~/.forward` as follows:
```
"|deliverf -n remotedir/Inbox"
```
Note that this requires that the users HOME directory always be available.

Locking / Mutex
===============
deliverf should interwork with movemail, email clients and other delivery agents because it locks the destination
file when it is writing to it. It does this with a dot-lock file and using Linux file locking (`flock`).

Issues
======
   * Mail may be deliverred as userid nobody.nogroup (depending on how your postfix, aliases etc are configured).
   * 

[backscatter]: http://en.wikipedia.org/wiki/Backscatter_(email)
