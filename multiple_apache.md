# Running two separate Apache servers on one machine

## This guide is for RHEL7 (Scientific Linux) mainly with systemd and SELinux

## Basic places

* /etc/httpd - the Apache server location
** /etc/httpd/conf - folder with httpd.conf files
** /etc/httpd/conf.d - folder with our own *.conf files
** /etc/httpd/conf.modules.d - folder with module *.conf files

* /usr/lib/systemd/system/httpd.service - service file location
* /etc/sysconfig/httpd - config file for the service

## STEP 1 - Creating new httpd config files

Copy the original httpd.conf file right next to it with desired name

```bash
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd-example.conf
```

Open the new conf file and you must definitely edit these:

After ServerRoot property add: `PidFile "/var/run/httpd-example.pid"`.
This will create new PID file for the new server. Otherwise it will fail to start.

Change the listening port

```
Listen 80 -> Listen 81
```

Change `ErrorLog` and `CustomLog` entries to its own new files.

```
CustomLog "logs/access_log" -> CustomLog "logs/access_log-example"
ErrorLog "logs/error_log" -> ErrorLog "logs/error_log-example"
```

If you wish to edit conf.modules.d or conf.d files create a copy of them and update the paths to them.
This is useful for example if you want to run multiple instances of WSGI.
```
Include conf.modules.d/*.conf -> Include conf.modules.d-example/*.conf
Include conf.d/*.conf -> Include conf.d-example/*.conf
```

## STEP 2 - New systemd service

Copy the httpd.service file.
```bash
cp /usr/lib/systemd/system/httpd.service /usr/lib/systemd/system/httpd-example.service
```

Open it and change these lines:
```
Description - so we know which one we are operating with
EnvironmentFile - change the path accordingly
```

Copy the sysconfig file.
```bash
cp /etc/sysconfig/httpd /etc/sysconfig/httpd-example
```

Uncomment the `OPTIONS` variable and add:
```
OPTIONS= -f conf/httpd-example.conf
```

This will load our edited conf file for httpd

## STEP 3 - SELinux
If you copied the conf.d/ or conf.modules.d/ folder you must tell SELinux they are safe to operate with.
```
chcon -R -u system_u -r object_r -t httpd_config_t conf.d conf.modules.d conf.d-example conf.modules.d-example
```

## STEP 4 - Wrapping Up
Now we can do some systemctl magic. We reload the systemctl daemon, enable our new server to start and start it.

```
systemctl --system daemon-reload
systemctl enable httpd-example
systemctl start httpd-example
```

Now you should be able to access your second instance of httpd!

### Acknowledgment
I am not responsible for any of your stupidy. If you break anything, it is your fault. You should have backups!

### References
I used this email thread which basically is this guide but with minor edits: https://lists.fedoraproject.org/pipermail/users/2013-August/439603.html
Config directives available in httpd.conf: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/3/html/Reference_Guide/s1-apache-config.html
