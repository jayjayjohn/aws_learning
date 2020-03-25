### Making an Amazon EBS Volume Available for Use on Linux
#### EBS is like a USB drive, you have to map/mount it to use. 
```
[ec2-user@ip-172-31-36-197 ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0  50G  0 disk
[ec2-user@ip-172-31-36-197 ~]$ sudo file -s /dev/xvdf
/dev/xvdf: data
[ec2-user@ip-172-31-36-197 ~]$ sudo mkdir /data

[ec2-user@ip-172-31-36-197 ~]$ sudo mkfs -t xfs /dev/xvdf
meta-data=/dev/xvdf              isize=512    agcount=4, agsize=3276800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=0
data     =                       bsize=4096   blocks=13107200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=6400, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[ec2-user@ip-172-31-36-197 ~]$ sudo mount /dev/xvdf /data
[ec2-user@ip-172-31-36-197 ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0  50G  0 disk /data

```
