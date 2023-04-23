<p><strong>Downloads manifest</strong></p>

<p>wget <a href="https://github.com/Vaskata84/Vaskata84/blob/master/NEW-ingress-v1.7.0/deploy.yaml">https://github.com/Vaskata84/Vaskata84/blob/master/NEW-ingress-v1.7.0/deploy.yaml</a> -O nginx-deploy.yaml<br />
replace &#39;Deployment&#39; with &#39;Daemonset&#39;<br />
replace &#39;NodePort&#39; with &#39;LoadBalancer&#39;</p>

<p>sed -i &#39;s/type: NodePort/type: LoadBalancer/&#39; nginx-deploy.yaml</p>

<p>sed -i &#39;s/kind: Deployment/kind: DaemonSet/&#39; nginx-deploy.yaml<br />
create NGINX objects</p>

<p>$ kubectl apply -f nginx-deploy.yaml</p>

<p>$ kubectl get Validatingwebhookconfigurations ingress-nginx-admission NAME WEBHOOKS AGE ingress-nginx-admission 1 2m<br />
and finally disable the nginx validating webhook for kubernetes<br />
#Install App<br />
snap install yq<br />
kubectl get Validatingwebhookconfigurations ingress-nginx-admission -o=yaml | yq &#39;.webhooks[].failurePolicy = &quot;Ignore&quot;&#39; | kubectl apply -f - validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission configured</p>

<p>kubectl get all -n ingress-nginx</p>

<p><strong>Enable Secondary Ingress</strong></p>

<p># apply DaemonSet that creates secondary ingress<br />
wget <a href="https://github.com/Vaskata84/Vaskata84/blob/master/NEW-ingress-v1.7.0/nginx-ingress-secondary.yaml">https://github.com/Vaskata84/Vaskata84/blob/master/NEW-ingress-v1.7.0/nginx-ingress-secondary.yaml</a> -O nginx-ingress-secondary-controller.yaml</p>

<p># set namespace<br />
sed -i &#39;s/{{nginx_ns}}/ingress-nginx/&#39; nginx-ingress-secondary-controller.yaml</p>

<p>$ kubectl apply -f nginx-ingress-secondary-controller.yaml<br />
daemonset.apps/nginx-ingress-secondary-controller created</p>

<p>Then create the secondary ingress service</p>

<p>$ wget <a href="https://github.com/Vaskata84/Vaskata84/blob/master/NEW-ingress-v1.7.0/nginx-ingress-secondary-service.yaml">https://github.com/Vaskata84/Vaskata84/blob/master/NEW-ingress-v1.7.0/nginx-ingress-secondary-service.yaml</a> -O nginx-ingress-secondary-service.yaml</p>

<p># set namespace and second MetalLB-IP<br />
$ sed -i &#39;s/{{nginx_ns}}/ingress-nginx/&#39; nginx-ingress-secondary-service.yaml<br />
$ sed -i &#39;s/loadBalancerIP: .*/loadBalancerIP: 192.168.88.241/&#39; nginx-ingress-secondary-service.yaml</p>

<p>$ kubectl apply -f nginx-ingress-secondary-service.yaml<br />
service/ingress-secondary created</p>

<p>Now you can see the secondary NGINX Daemonset and Services.</p>

<p># see primary and secondary nginx-controller<br />
$ kubectl get ds -n ingress-nginx<br />
NAME DESIRED CURRENT READY UP-TO-DATE AVAILABLE NODE SELECTOR AGE<br />
ingress-nginx-controller 1 1 1 1 1 kubernetes.io/os=linux 34m<br />
nginx-ingress-secondary-controller 1 1 1 1 1 &lt;none&gt; 1m15s</p>

<p>$ kubectl get services -n ingress-nginx</p>

<p>&nbsp;</p>

<pre>
NAME                                 TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
ingress-nginx-controller-admission   ClusterIP      10.43.154.160                   443/TCP                      11m
ingress-nginx-controller             LoadBalancer   10.43.43.111   192.168.88.240   80:32652/TCP,443:31400/TCP   11m
ingress-secondary                    LoadBalancer   10.43.33.10   192.168.88.241    80:30113/TCP,443:30588/TCP   36s</pre>
