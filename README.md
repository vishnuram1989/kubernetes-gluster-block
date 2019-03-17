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

``` bash
Step 1 : Update Gluster Node details in Heketi API Server

cat << EOF >> /etc/hosts
<host-ip-1> <hostname-01>
<host-ip-2> <hostname-02>
<host-ip-2> <hostname-03>
EOF

Step 2 :  yum install heketi -y

Step 3 :  Genetate SSH-Keyen to athenticate gluster nodes via heketi

ssh-keygen -f /etc/heketi/heketi_key -t rsa -N ''
ssh-copy-id -i /etc/heketi/heketi_key.pub root@gluter-node1
ssh-copy-id -i /etc/heketi/heketi_key.pub root@gluter-node1
ssh-copy-id -i /etc/heketi/heketi_key.pub root@gluter-node1

Note : Less priviledged user can be used that should have priviledge to run gluster and lvm commands 

Step 4: wget https://github.com/heketi/heketi/releases/download/v8.0.0/heketi-v8.0.0.linux.amd64.tar.gz && tar xvzf heketi-client-v8.0.0.linux.amd64.tar.gz --strip=2 heketi-client/bin/heketi-cli 

Step 5 : mv heketi-cli /usr/bin/ && chmod +x /usr/bin/heketi-cli

Step 6 : COPY Heketi {topology.json and heketi.json} to /etc/heketi
cp Heketi{heketi.json,topology.json} /etc/heketi/

Note : 
  For JWT Activation  change "use_auth": true in heketi.json
  change ssh details as per your  environment in heketi.json
  "sshexec": {
      "keyfile": "<private-key-location>",
      "user": "<user-name>",
      "port": "22",
      "fstab": "/etc/fstab"
  }
  update topology files based on the gluster-nodes and devices attached to your system

```

## Heketi-API service enabling

``` bash
$ systemctl enable heketi && systemctl start heketi
$ heketi-cli topology load --json=/etc/heketi/template.json

Note : IF Everything goes fine heketi will format the  disk and add them to gluster in brick format
```

## Heketi Kubernetes Integration

Step 1 : update heketiapi url and other parametes in heketi-block.yaml
ex: resturl: "http://10.139.224.106:8080"
ex: volumenameprefix: "myblock"
ex: hacount: "1" # How many replicas you need
ex: chapauthenabled: "false"# If ChapAuthentication Needed
ex: opmode: "heketi"

Step 2 : Kubetcl apply -f kubernetes/