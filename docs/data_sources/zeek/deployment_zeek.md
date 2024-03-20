# Deployment of Zeek

## Zeek packages
Zeek packages are available from the main [zeek website](https://zeek.org/get-zeek/), [OpenSUSE Build System](https://build.opensuse.org/project/show/security:zeek) (not EL8/9) and from CERN for EL9.

### CERN repos

The following repositories are compatible with Rocky9/Alma9 and RHEL9

- https://linuxsoft.cern.ch/repos/zeek9-qa/ **recommended**
- https://linuxsoft.cern.ch/repos/zeek9-stable/ **still in testing**
- https://linuxsoft.cern.ch/repos/zeek9-testing/

### Example repo file based on CERN repos (Alma9/Rocky9)

`/etc/yum.repos.d/cern-zeek.repo`

```
[cern-zeek]
name=r9zeek
baseurl=https://linuxsoft.cern.ch/repos/zeek9-qa/$basearch/os/
enabled=1
gpgcheck=1
priority=1
```

**Add gpg key**

```
wget https://linuxsoft.cern.ch/repos/RPM-GPG-KEY-kojiv2
rpm --import RPM-GPG-KEY-kojiv2
```

## Installation (AlmaLinux9)

`python3-semantic_version-2.8.4-7` is required which can be installed via the Alma9 repo.

Install commands for Alma9 are then

```
yum install python3-semantic_version-2.8.4-7 zeek
```

## Installation (Rocky9)

`python3-semantic_version-2.8.4-7` is required which can be downloaded directly:

```
wget  https://repo.almalinux.org/development/almalinux/9/devel/noarch/Packages/python3-semantic_version-2.8.4-7.el9.noarch.rpm
```

Note that Rocky9 also needs `libpcap-devel` which is only available in [crb](https://wiki.rockylinux.org/rocky/repo/#notes-on-crb)

Install commands for Rocky9 are then

```
yum localinstall python3-semantic_version-2.8.4-7.el9.noarch.rpm
dnf config-manager --set-enabled crb

yum clean all

yum install zeek
```

## Sample environment variable script

`/etc/profile.d/zeek.sh`

```
#!/bin/bash
# General environment variables
export PATH=/opt/zeek/bin${PATH:+:${PATH}}
export MANPATH=/opt/zeek/share/man:${MANPATH}
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/zeek/lib
```
