# gluster-block

## This is a implementation for kubernetes Block Storage using gluster-block

# Pre-Requesties 
## This setup is based on a 3 node gluster server

```
Step 1 : update host files with glusterfs Node details

cat << EOF >> /etc/hosts
<host-ip-1> <hostname-01>
<host-ip-2> <hostname-02>
<host-ip-2> <hostname-03>
EOF

Step 2 : GlusterFS Package Requirements

yum install centos-release-gluster  -y

yum install glusterfs glusterfs-cli glusterfs-libs glusterfs-server -y
systemctl enable glusterd
systemctl start glusterd

yum install libtcmu tcmu-runner tcmu-runner-handler-glfs gluster-block -y

systemctl enable gluster-blockd
systemctl start gluster-blockd

```