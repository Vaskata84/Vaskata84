<p><strong>Downloads manifest</strong></p>

<p>wget <a href="https://github.com/Vaskata84/Vaskata84/blob/master/NEW-ingress-v1.10.1/deploy.yaml">https://github.com/Vaskata84/Vaskata84/blob/master/NEW-ingress-v1.7.0/deploy.yaml</a> -O nginx-deploy.yaml<br />
replace &#39;Deployment&#39; with &#39;Daemonset&#39;<br />
replace &#39;NodePort&#39; with &#39;LoadBalancer&#39;</p>

<p>sed -i &#39;s/type: NodePort/type: LoadBalancer/&#39; nginx-deploy.yaml</p>

<p>sed -i &#39;s/kind: Deployment/kind: DaemonSet/&#39; nginx-deploy.yaml<br />
create NGINX objects</p>

<p>$ kubectl apply -f nginx-deploy.yaml</p>

<p>$ kubectl get Validatingwebhookconfigurations ingress-nginx-admission<p>
<p>NAME WEBHOOKS AGE ingress-nginx-admission 1 2m<br />
and finally disable the nginx validating webhook for kubernetes<br />
#Install App<br />
snap install yq<br />
kubectl get Validatingwebhookconfigurations ingress-nginx-admission -o=yaml | yq &#39;.webhooks[].failurePolicy = &quot;Ignore&quot;&#39; | kubectl apply -f -<p> 
<p>validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission configured</p>

<p>kubectl get all -n ingress-nginx</p>



