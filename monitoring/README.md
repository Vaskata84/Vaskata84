<p><strong>Quick instruction on how to deploy Uptime-Kuma on your Kubernetes cluster.</strong></p>
<p>Uptime-Kuma is an easy to deploy and easy to use monitoring tool, that has several &ldquo;Monitoring Types&rdquo; like HTTP(s), port-checking, DNS and some more. Also it includes some real nice alerting mechanisms as Mail, Teams and Gotify. If you want to find out more about Uptime-Kuma</p>



# Вижте всички PV-та, свързани с nfs-client
kubectl get pv | grep nfs-client

# Вижте StorageClass
kubectl get storageclass nfs-client -o yaml

# Вижте всички PVC-та в кластера
kubectl get pvc --all-namespaces

# Вижте NFS provisioner pod детайли
kubectl describe pod nfs-subdir-external-provisioner-6f86f4bcfc-lz78s
