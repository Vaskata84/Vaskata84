<p><strong>MetalLB installation by manifest section, install into the metallb-system</strong><br />
# Download manifest<br />
wget https://github.com/Vaskata84/Vaskata84/blob/master/NEWmetallb/metallb-native.yaml</p>

<p># updating validatingwebhookconfigurations so it does not fail under k3s<br />
sed -i &#39;s/failurePolicy: Fail/failurePolicy: Ignore/&#39; metallb-native.yaml</p>

<p># apply manifest<br />
kubectl apply -f metallb-native.yaml</p>

<p># show ip address, including ens4 and ens5 which are used for MetalLB endpoints<br />
ip a</p>

<p># get MetalLB configmap template<br />
wget https://github.com/Vaskata84/Vaskata84/blob/master/NEWmetallb/metallb-ipaddresspool.yml -O metallb-ipaddresspool.yml</p>

<p># change addresses to MetalLB endpoints<br />
sed -i &#39;s/{{metal_lb_primary}}-{{metal_lb_secondary}}/192.168.88.240-192.168.88.250/&#39; metallb-ipaddresspool.yml</p>

<p># apply<br />
kubectl apply -f metallb-ipaddresspool.yml</p>

<p># show created objects<br />
$ kubectl get all -n metallb-system</p>

<p>&nbsp;</p>
