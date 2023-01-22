
Configure MetalLB to use additional NIC

Per the MetalLB installation by manifest section, install into the metallb-system namespace.

kubectl apply -f https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/namespace.yaml kubectl apply -f https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/metallb.yaml

Then create a MetalLB configmap that instructs it which IP to use.

Get MetalLB configmap template. wget https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/metal-LB.yaml -O metal-temp.yml sed -i 's/{{metal_lb_primary}}-{{metal_lb_secondary}}/192.168.88.240-192.168.88.250/' metal-temp.yml kubectl apply -f metal-teml.yml kubectl get all -n metallb-system

NAME READY STATUS RESTARTS AGE pod/controller-1f23s353fd-hfs34 1/1 Running 0 11s pod/speaker-4552f 1/1 Running 0 11s pod/speaker-hfsd4 1/1 Running 0 11s pod/speaker-gfr54 1/1 Running 0 11s

NAME DESIRED CURRENT READY UP-TO-DATE AVAILABLE NODE SELECTOR AGE daemonset.apps/speaker 3 3 3 3 3 kubernetes.io/os=linux 11s

NAME READY UP-TO-DATE AVAILABLE AGE deployment.apps/controller 1/1 1 1 11s

NAME DESIRED CURRENT READY AGE replicaset.apps/controller-1f23s353fd 1 1 1 11s

Downloads baremetal ingress-nginx wget https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/ingress-nginx/ingress-nginx-1.1.1v.yaml -O nginx-deploy.yaml

replace 'Deployment' with 'Daemonset' replace 'NodePort' with 'LoadBalancer' sed -i 's/type: NodePort/type: LoadBalancer/' nginx-deploy.yaml sed -i 's/kind: Deployment/kind: DaemonSet/' nginx-deploy.yaml
