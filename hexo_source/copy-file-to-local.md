---
title: How to copy file to local system?
categories:
- articles
tags:
- linux
date: 2018/12/08 18:42:07
comments: true
---

##Use nc(NetCat) Command （Not Security）
**Server side**
```
cat backup.iso | pv -b > nc -l 3333
```
**Client side**
```
nc 192.168.0.1 3333 | pv -b > backunp.iso
```
**ps: nc also can use for port scanning**
```
#It may be useful to know which ports are open and running services on a target machine. the -z flag can be used to tell nc to report open ports,ranther than initiate a connection.

nc -zv host.example.com 80 20 22

>nc: connect to host.example.com 80 (tcp) failed: Connection refused
>nc: connect to host.example.com 20 (tcp) failed: Connection refused
>Connection to host.example.com port [tcp/ssh] succeeded!

#nc also be useful to know which server software is running
echo 'QUIT' | nc host.example.com 20-30

>SSH-1.99-OpenSSH_3.6.1p2 Protocol mismatch.
>220 host.example.com IMS SMTP Receiver Version 0.84 Ready

```
##Use scp Command
**Copy your file to target marchine**
```
scp file.foo user@target.com:file.foo
```
**Copy target marchine's file to local**
```
scp user@target.com:file.foo file.foo
```
##Use ssh Command
**To remote host**
```
cat localfile.conf | ssh user@hostname 'cat -> /tmp/file.conf'
```
**From remote host**
```
ssh user@hostname 'cat /tmp/file.conf' > /tmp/file.conf
```
##Use sftp Command
**same as ftp command**
```
sftp username@remote_hostname_or_IP
```
##Use sshsf Mount Remote File System Over SSH
`scp sftp nc allow us to copy files easily and securely between these accounts, But, what if we don't want to copy the files to our local system before using them? Normally, this would be a good place for traditional network filesystem, Unfortunately, setting up these network filesystems requires administrator access on both systems. Luckily, as long as you have SSH access, you can use SSHFS to mount and use remote directory trees as if they were local.`
**Install sshfs**
```
sudo apt-get install sshfs
```
**Mounting the remote file system**
```
cd $HOME
sudo mkdir /mnt/droplte < -- replace 'droplet' whatever you prefer
sudo sshfs -o allow_other, defer_permissions root@remotehost:/ /mnt/droplet
```
**Unmounting the remote file system**
```
sudo umount /mnt/droplet
```
**Permanently mounting the remote file system**
`This would set a mount point that would persist through restart of both your local machine and droplets`
```
sudo nano /etc/fstab <-- edit the /etc/fstab file with a text editor
sshfs#root@remotehost:/ /mnt/droplet
```


