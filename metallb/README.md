<p style="text-align:start"><strong>Configure MetalLB to use additional NIC</strong></p>

<p style="text-align:start"><strong>Per the MetalLB installation by manifest section, install into the metallb-system namespace.</strong></p>

<p style="text-align:start"><span style="font-size:11px">kubectl apply -f&nbsp;<a href="https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/namespace.yaml" rel="nofollow" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none;">https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/namespace.yaml</a>&nbsp;</span></p>

<p style="text-align:start"><span style="font-size:11px">kubectl apply -f&nbsp;<a href="https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/metallb.yaml" rel="nofollow" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none;">https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/metallb.yaml</a></span></p>

<p style="text-align:start"><strong>Then create a MetalLB configmap that instructs it which IP to use.</strong></p>

<p style="text-align:start">Downloads MetalLB configmap template.</p>

<p style="text-align:start"><span style="font-size:11px">wget&nbsp;<a href="https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/metal-LB.yaml" rel="nofollow" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none;">https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/metallb/metal-LB.yaml</a>&nbsp;-O metal-temp.yml</span></p>

<p style="text-align:start"><span style="font-size:11px">sed -i &#39;s/{{metal_lb_primary}}-{{metal_lb_secondary}}/192.168.88.240-192.168.88.250/&#39; metal-temp.yml</span></p>

<p style="text-align:start"><span style="font-size:11px">kubectl apply -f metal-teml.yml</span></p>

<p style="text-align:start"><span style="font-size:11px">kubectl get all -n metallb-system</span></p>

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

<p style="text-align:start"><span style="font-size:11px">wget&nbsp;<a href="https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/ingress-nginx/ingress-nginx-1.1.1v.yaml" rel="nofollow" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none;">https://raw.githubusercontent.com/Vaskata84/Vaskata84/master/ingress-nginx/ingress-nginx-1.1.1v.yaml</a>&nbsp;-O nginx-deploy.yaml</span></p>

<p style="text-align:start"><span style="font-size:11px">replace &#39;Deployment&#39; with &#39;Daemonset&#39; replace &#39;NodePort&#39; with &#39;LoadBalancer&#39;</span></p>

<p style="text-align:start"><span style="font-size:11px">sed -i &#39;s/type: NodePort/type: LoadBalancer/&#39; nginx-deploy.yaml 
  sed -i &#39;s/kind: Deployment/kind: DaemonSet/&#39; nginx-deploy.yaml</span></p>

<pre>
<span style="font-size:11px">kubectl apply -f nginx-deploy.yaml
</span>
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
daemonset.apps/ingress-nginx-controller created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created</pre>

<pre>
<span style="font-size:11px">kubectl get all -n ingress-nginx</span></pre>

<pre>
NAME                                       READY   STATUS              RESTARTS   AGE
pod/ingress-nginx-controller-th54d         0/1     ContainerCreating   0          55s
pod/ingress-nginx-controller-fg45s         0/1     ContainerCreating   0          55s
pod/ingress-nginx-controller-drd33         0/1     ContainerCreating   0          55s
pod/ingress-nginx-admission-create-hg543   0/1     Completed           0          55s
pod/ingress-nginx-admission-patch-gh543    0/1     Completed           0          55s

NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)                    AGE
service/ingress-nginx-controller-admission   ClusterIP      10.43.154.43                  443/TCP                    55s
service/ingress-nginx-controller             LoadBalancer   10.43.27.211   192.168.88.240 80:32652/TCP,443:31400/TCP 55s

NAME                                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE  NODE SELECTOR          AGE
daemonset.apps/ingress-nginx-controller   3         3         0       3            0          kubernetes.io/os=linux 55s

NAME                                       COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   1/1           12s        55s
job.batch/ingress-nginx-admission-patch    1/1           13s        55s</pre>
