# gluster-block

## This is a implementation for kubernetes Block Storage using gluster-block

# Pre-Requesties 
```
1.  3 node gluster server
2.  Heketi-API Server
3.  Kubernetes Setup
```

Install
-------

GlusterFS and GlusterBlock has to be installed on gluster nodes

Add host information on /etc/hosts for all glusternodes

```
cat << EOF >> /etc/hosts
<host-ip-1> <hostname-01>
<host-ip-2> <hostname-02>
<host-ip-2> <hostname-03>
EOF
```

Install Dependencies

```bash
yum install centos-release-gluster  -y

yum install glusterfs glusterfs-cli glusterfs-libs glusterfs-server -y

yum install libtcmu tcmu-runner tcmu-runner-handler-glfs gluster-block -y

```
Enable and Start Gluster Services

```bash
systemctl enable glusterd
systemctl start glusterd
systemctl enable gluster-blockd
systemctl start gluster-blockd

```

#### Heketi-API Node setup


 Update Gluster Node details in Heketi API Server
``` bash
cat << EOF >> /etc/hosts
<host-ip-1> <hostname-01>
<host-ip-2> <hostname-02>
<host-ip-2> <hostname-03>
EOF
```

Install heketi dependencies

```bash
yum install heketi -y
```

Generate SSH-Keyen to athenticate gluster nodes via heketi

```bash
ssh-keygen -f /etc/heketi/heketi_key -t rsa -N ''
ssh-copy-id -i /etc/heketi/heketi_key.pub root@gluter-node1
ssh-copy-id -i /etc/heketi/heketi_key.pub root@gluter-node1
ssh-copy-id -i /etc/heketi/heketi_key.pub root@gluter-node1

Note : Less priviledged user can be used that should have priviledge to run gluster and lvm commands 
```

Install Heketi-cli to access Heketi API server

```bash
 wget https://github.com/heketi/heketi/releases/download/v8.0.0/heketi-v8.0.0.linux.amd64.tar.gz && tar xvzf heketi-client-v8.0.0.linux.amd64.tar.gz 
  cp heketi/heketi-client/bin/heketi-cli  /usr/bin && chmod +x /usr/bin/heketi
```

There are several parameters in Heketi {topology.json and heketi.json} before starting the heketi service


Heketi JWT option for security in heketi.json

```json
  "use_auth": false,
```

For Executor type SSH|Mock
```json
"executor": "ssh",
"sshexec": {
      "keyfile": "<private-key-location>",
      "user": "<user-name>",
      "port": "22",
      "fstab": "/etc/fstab"
  }

for Block Storage activation 
```json

   "auto_create_block_hosting_volume": true,  
   "block_hosting_volume_size": <size-in-GB>
```

Start Heketi-API service

``` bash
 systemctl enable heketi && systemctl start heketi
```

Topology.json defines our gluster configuration. Modify as per the gluster config. Refer heketi/topology.json for entire configuration

```json

 {
                    "node": {
                        "hostnames": {
                            "manage": [
                                "<host-01>"
                            ],
                            "storage": [
                                "<host-ip-01>"
                            ]
                        },
                        "zone": 1
                    },
                    "devices": [
                        {
                            "name": "/dev/sda"
                        }
                    ]
                }
```

Load topology.json to heketi 

```bash
heketi-cli topology load --json=/etc/heketi/template.json

Note: This will format the disk and prepare for gluster use. This removes manual maintanence of strage from the backend
IF Everything goes fine heketi will format the  disk and add them to gluster in brick format
```


## Kubernetes client packages installation
```bash
yum install iscsi-initiator-utils
service iscsid start
service iscsid status
```


## Heketi Kubernetes Integration

Heketi secret creation if JWT Enabled

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: heketi-secret
  namespace: default
# echo -n "password" | base64
data:
  key: <password-value>
type: gluster.org/glusterblock
```

kubectl apply -f kubernetes/heketi-secret.yaml

Heketi Storage Class creation on kubernetes

```
parameters:
  resturl: "http://<ip-address>:<port>"
  restuser: "<user-name>"
  restsecretNamespace: "default"
  restsecretname: "heketi-secret"
  opmode: "heketi"
  hacount: "1"
  chapauthenabled: "false"
  volumenameprefix: "myblock"

Note: modify parameters as per the need
```
kubectl apply -f kubernetes/heketiapi-block.yaml

### External storage provisioner configuration for GlusterBlock
Create a service account to use with storage provisioner

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: glusterblock-provisioner`
```

kubectl apply -f kubernetes/blockstorage-sa.yaml

Create RBAC with priveledges required for provisioner 

kubecl apply -f kubernetes/rbac.yaml

External Provisioner Configuration
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: glusterblock-provisioner
spec:
  serviceAccountName: glusterblock-provisioner
  containers:
    - image: "quay.io/external_storage/glusterblock-provisioner:latest"
      name: glusterblock-provisioner
      env:
        - name: PROVISIONER_NAME
          value: "gluster.org/glusterblock"
```
kubectl apply -f kubernetes/provisioner.yaml

##Block Storage demo
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: demo-pvc
  annotations:
    volume.beta.kubernetes.io/storage-class: "gluster-block"
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gb
```
kubectl apply -f kubernetes/demo-pvc.yaml

