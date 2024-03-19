# Installation

## Daemon

For ubuntu systems: https://www.syslog-ng.com/community/b/blog/posts/installing-the-latest-syslog-ng-on-ubuntu-and-other-deb-distributions

Make sure to use `ubuntu-jammy` for latest ubuntu lts.

## Docker (not really convenient IMHO)

Not really understanding how to make it work, although seems updated https://github.com/linuxserver/docker-syslog-ng

# Version Checking
Check version:

```bash
❯ syslog-ng -V    
syslog-ng 4 (4.6.0)
Config version: 4.2
Installer-Version: 4.6.0
Revision: 4.6.0-1
Compile-Date: Jan  9 2024 12:16:14
Module-Directory: /usr/lib/syslog-ng/4.6
Module-Path: /usr/lib/syslog-ng/4.6
Include-Path: /usr/share/syslog-ng/include
Available-Modules: cryptofuncs,rate-limit-filter,correlation,hook-commands,confgen,afprog,cef,affile,tags-parser,afsocket,basicfuncs,disk-buffer,syslogformat,metrics-probe,linux-kmsg-format,kvformat,timestamp,pacctformat,appmodel,regexp-parser,system-source,afuser,json-plugin,csvparser,pseudofile,sdjournal
Enable-Debug: off
Enable-GProf: off
Enable-Memtrace: off
Enable-IPv6: on
Enable-Spoof-Source: on
Enable-TCP-Wrapper: on
Enable-Linux-Caps: on
Enable-Systemd: on
```

# Testing only via files

Start `syslog-ng` in the **foreground** using a simple custom config.

```bash
sudo syslog-ng -Fvde -f syslog-ng.conf
```

> - -s does syntax checking and helps you spot configuration errors before a configuration goes live. Note that it cannot find all problems, like a typo in a source name.
> - -F starts syslog-ng in the foreground
> - -vde provides you with extra information on the terminal about what syslog-ng is doing
> - -f path/to/config allows you to use an alternate configuration instead of the default.


Let it run, and in another terminal write a custom message:

```bash
logger this is a test
tail /var/log/messages
```

You should see the written messages in both terminals:

```
Mar 19 12:10:44 sergioi2cat sergiogimenez[71076]: this is a test
```

And in syslog-ng stdout:

```
[2024-03-19T12:14:17.083627] Outgoing message; message='Mar 19 12:14:16 sergioi2cat sergiogimenez[72552]: this is a test\x0a'
```

# Checking `syslog-ng.conf` syntax

Using the `-s` flag we can check if the conf file is syntactically correct:

```bash
syslog-ng -s -f ./syslog-ng.conf
```

No output means no problem found.

# Testing Network Syslog Server

The following syslog-ng configuration implements a very simple syslog-ng server. Save this as netsource.conf on your test machine.

```
@version:3.38
source s_tcp { tcp(port(514)); };
destination d_file { file("/tmp/fromnet"); };
log { source(s_tcp); destination(d_file); };
```

Stop the running syslog implementation and start syslog-ng with this configuration in the foreground with debug information enabled:

```bash
sudo syslog-ng -Fvde -f ./syslog-ng.conf
```

From another terminal, you can now use the logger command to generate test messages. Note that the exact parameters of logger might be different on your system:

```bash
logger -T -n 127.0.0.1 -P 514 this is a test message
```

> - -T: TCP
> - -n: hostname or IP
> - -P: port
> - Log message

## Same test but UDP

```
@version:3.38
source s_udp { udp(port(514)); };
destination d_file { file("/tmp/fromnet"); };
log { source(s_udp); destination(d_file); };
```

Test command:

```bash
logger  -n 127.0.0.1 -P 514 "this is a test message"
```
> Same as before, but if we do not specify the `-T` flag by default uses UDP.

