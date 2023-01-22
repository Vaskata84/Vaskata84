<p style="text-align:start"><strong>Configure MetalLB to use additional NIC</strong></p>

<p style="text-align:start"><strong>Per the MetalLB installation by manifest section, install into the metallb-system namespace.</strong></p>

<p style="text-align:start">kubectl apply -f&nbsp;<a href="https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/namespace.yaml" rel="nofollow" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none;">https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/namespace.yaml</a>&nbsp;</p>

<p style="text-align:start">kubectl apply -f&nbsp;<a href="https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/metallb.yaml" rel="nofollow" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none;">https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/metallb.yaml</a></p>

<p style="text-align:start"><strong>Then create a MetalLB configmap that instructs it which IP to use.</strong></p>

<p style="text-align:start">Downloads MetalLB configmap template.</p>

<p style="text-align:start">wget&nbsp;<a href="https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/metal-LB.yaml" rel="nofollow" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none;">https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/metal-LB.yaml</a>&nbsp;-O metal-temp.yml</p>

<p style="text-align:start">sed -i &#39;s/{{metal_lb_primary}}-{{metal_lb_secondary}}/192.168.88.240-192.168.88.250/&#39; metal-temp.yml</p>

<p style="text-align:start">kubectl apply -f metal-teml.yml</p>

<p style="text-align:start">kubectl get all -n metallb-system</p>

<pre>
NAME                              READY   STATUS    RESTARTS   AGE
pod/controller-4hdjf65hw4-rb7dc   1/1     Running   0          11s
pod/speaker-56gdd                 1/1     Running   0          11s
pod/speaker-qwsa3                 1/1     Running   0          11s
pod/speaker-4red5                 1/1     Running   0          11s

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   3         3         3       3            3           kubernetes.io/os=linux   11s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           11s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-4hdjf65hw4   1         1         1       11s</pre>

<p style="text-align:start"><strong>Downloads baremetal ingress-nginx </strong></p>

<p style="text-align:start">wget&nbsp;<a href="https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/ingress-nginx/ingress-nginx-1.1.1v.yaml" rel="nofollow" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none;">https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/ingress-nginx/ingress-nginx-1.1.1v.yaml</a>&nbsp;-O nginx-deploy.yaml</p>

<p style="text-align:start">replace &#39;Deployment&#39; with &#39;Daemonset&#39; replace &#39;NodePort&#39; with &#39;LoadBalancer&#39;</p>

<p style="text-align:start">sed -i &#39;s/type: NodePort/type: LoadBalancer/&#39; nginx-deploy.yaml sed -i &#39;s/kind: Deployment/kind: DaemonSet/&#39; nginx-deploy.yaml</p>
