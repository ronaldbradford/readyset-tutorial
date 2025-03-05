# Readyset Server

These instructions apply to a running ReadySet server on an Debian system. See [Install Readyset System Packages](https://readyset.io/docs/get-started/install-rs/binaries/install-package) for more information.

The output in this tutorial is from an Ubuntu 22.04 installation.

```
$ uname -a

Linux picard 6.8.0-51-generic #52~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Mon Dec  9 15:00:52 UTC 2 x86_64 x86_64 x86_64 GNU/Linux

$ cat /etc/os-release

PRETTY_NAME="Ubuntu 22.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.1 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```

## Starting the service

```
$ sudo systemctl start readyset
```

## Stopping the service

```
$ sudo systemctl stop readyset
```


## Checking the status of the service
```
$ sudo systemctl status readyset

● readyset.service - ReadySet standalone daemon
     Loaded: loaded (/lib/systemd/system/readyset.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2025-03-01 22:51:01 EST; 10h ago
   Main PID: 3783216 (readyset)
      Tasks: 99 (limit: 76749)
     Memory: 837.6M
        CPU: 14min 38.118s
     CGroup: /system.slice/readyset.service
             └─3783216 /usr/bin/readyset

Mar 01 22:51:01 picard systemd[1]: Started ReadySet standalone daemon.
```

## Readyset logging

By default, daily logs are stored in the `/var/lib/readyset` directory.

```
$ ls -lh /var/lib/readyset/*log*

-rw-r--r-- 1 readyset readyset 143K Feb 27 16:45 readyset.log.2025-02-27
-rw-r--r-- 1 readyset readyset 8.9K Feb 28 18:51 readyset.log.2025-02-28
-rw-r--r-- 1 readyset readyset  68K Mar  1 18:59 readyset.log.2025-03-01
-rw-r--r-- 1 readyset readyset  13K Mar  2 10:18 readyset.log.2025-03-02
```

This logging is configurable via the `LOG_PATH` and `LOG_ROTATION variables, which can be fond in the Readyset configuration file, generally found in `/etc/readyset/readyset.conf`.

```
$ sudo grep ^LOG  /etc/readyset/readyset.conf

LOG_PATH=/var/lib/readyset
LOG_ROTATION=daily
```


