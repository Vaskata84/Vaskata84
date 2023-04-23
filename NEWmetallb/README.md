MetalLB installation by manifest section, install into the metallb-system 
# download manifest
wget https://github.com/Vaskata84/Vaskata84/blob/master/NEWmetallb/metallb-native.yaml

# updating validatingwebhookconfigurations so it does not fail under k3s
sed -i 's/failurePolicy: Fail/failurePolicy: Ignore/' metallb-native.yaml

# apply manifest
kubectl apply -f metallb-native.yaml

# show ip address, including ens4 and ens5 which are used for MetalLB endpoints
ip a

# get MetalLB configmap template
wget https://github.com/Vaskata84/Vaskata84/blob/master/NEWmetallb/metallb-ipaddresspool.yml -O metallb-ipaddresspool.yml

# change addresses to MetalLB endpoints
sed -i 's/{{metal_lb_primary}}-{{metal_lb_secondary}}/192.168.88.240-192.168.88.250/' metallb-ipaddresspool.yml

# apply
kubectl apply -f metallb-ipaddresspool.yml

# show created objects
$ kubectl get all -n metallb-system
