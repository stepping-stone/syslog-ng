# Overview
Modular [syslog-ng](http://www.balabit.com/network-security/syslog-ng) configuration, with one log file per daemon or service.

The log files are stored below <code>/var/log/syslog/\<daemon\>.log</code>
Log messages which aren't associated with a specific service or daemon will be logged to the catch-all <code>/var/log/messages</code> log file.
For example, syslog-ng will create log messages as seen below:
```bash
ls /var/log/syslog
```
```
acpid.log       dhcpcd.log  lightdm.log         polkitd.log  sshd.log
bluetoothd.log  dhcpd.log   NetworkManager.log   portage.log  syslog-ng.log
dbus.log        kernel.log  ntpd.log            postfix.log
```

# Installation
Simply clone the Git repository and let <code>/etc/syslog-ng</code> point to the root of your cloned repository:

```bash
git clone https://github.com/stepping-stone/syslog-ng.git ~/repos/syslog-ng
mv /etc/syslog-ng /etc/syslog-ng.orig
ln -s ~/repos/syslog-ng /etc/syslog-ng
/etc/init.d/syslog-ng restart
```

If you're looking for a more enterprisish way to install the configuration, check out the [puppet-syslogng](https://github.com/purplehazech/puppet-syslogng) module, which is based on this configuration.

# Contribution
Contributions are very welcome, simply fork our repository and send us a pull-request. If you found a bug, open an issue.

## Missing daemon or service configuration
There are so many daemons out there, that we can't add all by ourself :) If you're using a software for which there is no configuration yet, proceed with the following basic steps to create and submit a new configuration:

1. Fork our repository on GitHub

2. Create the required configuration files:

```bash
serviceName=               # For example OpenSSH
serviceProgramName=        # For example sshd

# Create the syslog-ng filter 
cat << EOF > "syslog-ng.conf.d/filter.d/${serviceProgramName}.conf"
# ${serviceName} (${serviceProgramName}) filter

filter f_${serviceProgramName} { program("^${serviceProgramName}\$"); };
EOF

# Create the syslog-ng file destination
cat << EOF > "syslog-ng.conf.d/destination.d/${serviceProgramName}.conf"
# ${serviceName} (${serviceProgramName}) destination

destination d_${serviceProgramName} { file("\`syslog_dir\`/${serviceProgramName}.log"); };
EOF

# Create the syslog-ng default file log path
cat << EOF > "syslog-ng.conf.d/log.d/90_${serviceProgramName}.conf"
# ${serviceName} (${serviceProgramName}) final file log

log { source(s_log); filter(f_${serviceProgramName}); destination(d_${serviceProgramName}); flags(final); };
EOF

/etc/init.d/syslog-ng reload
```

3. Test your new config snippets, by generating a log message from your new software and see if <code>/var/log/syslog/\<serviceProgramName\>.log</code> gets created.

4. Commit and push your additions

```bash
git add syslog-ng.conf.d/filter.d/${serviceProgramName}.conf \
        syslog-ng.conf.d/destination.d/${serviceProgramName}.conf \
        syslog-ng.conf.d/log.d/90_${serviceProgramName}.conf

git commit -m "Adding configuration for ${serviceName} (${serviceProgramName})."

git push
```

5. Send us a pull-request.
6. Thank you! :)
