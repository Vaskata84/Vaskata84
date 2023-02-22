<main>
    <article>
        <header>
            <h1 itemprop="headline"><a href="https://fabianlee.org/2022/01/12/kubernetes-nfs-mount-using-dynamic-volume-and-storage-class/" rel="bookmark" title="Kubernetes: NFS mount using dynamic volume and Storage Class">Kubernetes: NFS mount using dynamic volume and Storage Class</a></h1>
        </header>
    </article>
</main>

<p><strong>Install NFS Server&nbsp;</strong></p>
<pre>sudo apt-get update
sudo apt-get install nfs-common nfs-kernel-server -y</pre>
<p><strong>Create directory to export</strong></p>
<pre>sudo mkdir -p /data/nfs1
sudo chown nobody:nogroup /data/nfs1
sudo chmod g+rwxs /data/nfs1</pre>
<pre># limit access to clients in 192.168/16 network
$ echo -e &quot;/data/nfs1\t192.168.0.0/16(rw,sync,no_subtree_check,no_root_squash)&quot; | sudo tee -a /etc/exports

$ sudo exportfs -av 
/data/nfs1 192.168.0.0/16</pre>
<pre>restart and show logs
sudo systemctl restart nfs-kernel-server
sudo systemctl status nfs-kernel-server</pre>
<pre># show for localhost
$ /sbin/showmount -e localhost
Export list for 127.0.0.1:
/data/nfs1 192.168.0.0/16

</pre>
<pre># on Debian/Ubuntu based nodes Must be installed on all nodes and workers
sudo apt update
sudo apt install nfs-common -y

</pre>
<pre># use default public IP of host 
$ /sbin/showmount -e 192.168.88.10 
Export list for 192.168.88.10: 
/data/nfs1 192.168.0.0/16


# show for default public IP of host
$ /sbin/showmount -e 192.168.88.10
Export list for 192.168.88.10: 
/data/nfs1 192.168.0.0/16</pre>


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

<pre>wget <a href="https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/pv-volume/pvc.yaml">https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/pv-volume/pvc.yaml</a>

# apply 
kubectl apply -f pvc.yaml

# display new pvc object 
$ kubectl get pvc sc-nfs-pvc
NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
sc-nfs-pvc   Bound    pvc-cc93ac54-60ds-4edf-9a48-fd4e1643sawf   2Gi        RWO            nfs-client     15s</pre>

