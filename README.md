# deliverf
Utility to support delayed delivery of Postfix mail to a file.
## Project Status
In (small) production. Seems to be working ok. Feedback welcome.
# Introduction
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
generate a separate non-delivery report (NDR) bounce message or [backscatter].

This simple script allows postfix to queue and re-attempt delivery of the message
if the filesystem is not available. It does this by returning an exit code of
75 (`EX_TEMPFAIL`) when it cannot lock and write the destination file.

# Usage
Most of the time, deliverf will be used with `procmail` or `~/.forward` as follows:
```
"|deliverf remotedir/Inbox"
```
Note that this requires that the users HOME directory always be available.

It is possible to invoke deliverf directly from aliases, however, this is
likely not the best way to use it because it will run with `nobody` permissions:
```
bob:    "|deliverf -s /mnt/users/bob/Inbox"
```
or
```
bob:    "|deliverf -d /mnt/users/bob/Inbox"
```
## Encrypted Home Directory
If the user wants their mail delivered to an encrypted directory that is mounted
when they log in (or some time after system boots), then in their *non-encrypted* HOME
directory, they could put a `.forward` which directs their mail to an encrypted
sub-directory which does not exist until their encrypted HOME is mounted. Provided
they mount their encrypted HOME directory before the mail queue times out (postfix defaults to 5 days),
their mail will be delivered into the encrypted home:
```
"|deliverf /home/user/private/Mail/Inbox"
```
 - you probably want to look at or test the behaviour of your mail queues in consideration
 of this.
 - Ensure time-outs are long enough and figure out if you would really want to bounce
 that mail, discard it, or deal with it in some other way.

# Locking / Mutex
deliverf should inter-work with movemail, email clients, and other delivery
agents because it locks
the destination file while writing. It does this with a dot-lock file and
advisory file locking (`fcntl` or `flock`).

# Logging
Diagnostic output goes to stderr and optionally to syslog mail.info
with `-s` or `--syslog`

# Issues
  * Mail may be delivered as user-id nobody.nogroup (depending on how your postfix, aliases etc are configured).
  * 

[backscatter]: http://en.wikipedia.org/wiki/Backscatter_(email)
