# gluster-block

## This is a implementation for kubernetes Block Storage using gluster-block

# Pre-Requesties 
```
yum install centos-release-gluster  -y

yum install glusterfs glusterfs-cli glusterfs-libs glusterfs-server -y
systemctl enable glusterd
systemctl start glusterd

yum install libtcmu tcmu-runner tcmu-runner-handler-glfs gluster-block -y

systemctl enable gluster-blockd
systemctl start gluster-blockd

```