# RHEL8 LibreNMS installation

## Install RHEL 8 in LibreNMS

Become super user

```
sudo -s
```

Update yum cache

```
dnf update -y
```

Check current Linux kernal

```
uname -r
```

Output of above command

```
5.4.17-2102.201.3.el8uek.x86_64
```

Check Redhat release

```
cat /etc/redhat-release
```

Output of above command

```
Red Hat Enterprise Linux release 8.5 (Ootpa)
```

Install EPEL (Extra Packages for Enterprise Linux)

```
dnf -y install epel-release
```

