1. Requirements

    Installed kubectl command-line tool.
    Have a kubeconfig file (default location is ~/.kube/config).
    CoreDNS. Can be enabled for microk8s by microk8s enable dns && microk8s stop && microk8s start



kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


2. Access The Argo CD API Server

By default, the Argo CD API server is not exposed with an external IP. To access the API server, choose one of the following techniques to expose the Argo CD API server:
Service Type Load Balancer

Change the argocd-server service type to LoadBalancer:

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

argocd admin initial-password

argocd login <ARGOCD_SERVER>

argocd account update-password

First list all clusters contexts in your current kubeconfig:

kubectl config get-contexts -o name

argocd cluster add docker-desktop

kubectl config set-context --current --namespace=argocd
