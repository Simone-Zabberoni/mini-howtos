# Encrypted filesystem on top of LVM

## Setup a LVM volume

**Important**: I'm working on the device directly, without a partition table. I'll be able to enalarge the VmWare disk, then the LVM without worrying about partitions.

```
[root@dummy ~]# pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
[root@dummy ~]# vgcreate encrypted /dev/sdc
  Volume group "encrypted" successfully created
[root@dummy ~]# lvcreate -l 100%FREE encrypted -n lv_encrypted
  Logical volume "lv_encrypted" created.
```


## Keyfile

Use a keyfile to mount and decrypt on boot.
Of course, it's less secure than inserting the passphrase manually at boot.

```
[root@dummy ~]# echo -n 'someStrongPassword' > /etc/keys/www_data
[root@dummy ~]# chown root:root /etc/keys/www_data
[root@dummy ~]# chmod 400 /etc/keys/www_data
```

## Encrypted filesystem creation

Create the encrypted fs on the logical volume `lv_encrypted` using the keyfile (or a password, if you omit the file):

```
[root@dummy ~]# cryptsetup luksFormat -c aes-xts-plain64 -s 512 /dev/encrypted/lv_encrypted /etc/keys/www_data

WARNING!
========
This will overwrite data on /dev/encrypted/lv_encrypted irrevocably.

Are you sure? (Type uppercase yes): YES
[...]
```

## Open and format it

Open the encrypted volume with the key file (or password) and bind it into the devmapper as `www_data`:
```
cryptsetup open /dev/encrypted/lv_encrypted www_data --key-file /etc/keys/www_data
```

Then format it:
```
[root@dummy ~]# mkfs.xfs /dev/mapper/www_data
```

## Open and mount at boot

Add to `/etc/crypttab` a definition line for the encrypted volume, with the devmapper binding:
```
www_data        /dev/encrypted/lv_encrypted     /etc/keys/www_data
```

Add to `/etc/fstab` a definition line for the devmapper entry:

```
# Encrypted www
/dev/mapper/www_data            /var/www/html           xfs     defaults  0  2
```




