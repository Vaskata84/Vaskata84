# downloads manifest
wget https://github.com/Vaskata84/Vaskata84/blob/master/NEW-ingress-v1.7.0/deploy.yaml -O nginx-deploy.yaml

# replace 'Deployment' with 'Daemonset'
# replace 'NodePort' with 'LoadBalancer'
sed -i 's/type: NodePort/type: LoadBalancer/' nginx-deploy.yaml
sed -i 's/kind: Deployment/kind: DaemonSet/' nginx-deploy.yaml

# create NGINX objects
$ kubectl apply -f nginx-deploy.yaml

$ kubectl get Validatingwebhookconfigurations ingress-nginx-admission
NAME                      WEBHOOKS   AGE
ingress-nginx-admission   1          2m

# and finally disable the nginx validating webhook for kubernetes
snap install yq 
kubectl get Validatingwebhookconfigurations ingress-nginx-admission -o=yaml | yq '.webhooks[].failurePolicy = "Ignore"' | kubectl apply -f -
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission configured

kubectl get all -n ingress-nginx
