<pre>helm install nfs-subdir-external-provisioner \
nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--set nfs.server=192.168.88.10 \
--set nfs.path=/data/nfs1 \
--set storageClass.onDelete=true

NAME: nfs-subdir-external-provisioner
LAST DEPLOYED: Tue Jan 22 10:58:16 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

$ kubectl get storageclass nfs-client
NAME         PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client   cluster.local/nfs-subdir-external-provisioner   Delete          Immediate           true                   10s</pre>


