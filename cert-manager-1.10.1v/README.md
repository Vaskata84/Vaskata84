https://cert-manager.io/docs/installation/
kubectl apply -f cert-manager.yaml

If you're still getting the cert-manager HTTP01 challenges failing error.

kubectl get deployments -n cert-manager
kubectl get deployment cert-manager -n cert-manager -o yaml hostNetwork:false to true :)
