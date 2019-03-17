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

## Heketi-API Node setup

``` 
Step 1 :  yum install heketi -y

Step 2 :  Genetate SSH-Keyen to athenticate gluster nodes via heketi

ssh-keygen -f /etc/heketi/heketi_key -t rsa -N ''
ssh-copy-id -i /etc/heketi/heketi_key.pub root@gluter-node1
ssh-copy-id -i /etc/heketi/heketi_key.pub root@gluter-node1
ssh-copy-id -i /etc/heketi/heketi_key.pub root@gluter-node1

Note : Less priviledged user can be used that should have priviledge to run gluster and lvm commands 

Step 3: wget https://github.com/heketi/heketi/releases/download/v8.0.0/heketi-v8.0.0.linux.amd64.tar.gz && tar xvzf heketi-client-v8.0.0.linux.amd64.tar.gz --strip=2 heketi-client/bin/heketi-cli 

Step 4 : mv heketi-cli /usr/bin/ && chmod +x /usr/bin/heketi-cli

Step 5 : COPY Heketi {topology.json and heketi.json} to /etc/heketi
cp Heketi{heketi.json,topology.json} /etc/heketi/

Note : 
  if JWT needed change "use_auth": true
  change ssh details as your the environment
  "sshexec": {
      "keyfile": "<private-key-location>",
      "user": "<user-name>",
      "port": "22",
      "fstab": "/etc/fstab"
  }
  update topology files based on the gluster-nodes and devices attached to your system

```