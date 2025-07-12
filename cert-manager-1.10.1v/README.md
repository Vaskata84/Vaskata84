https://cert-manager.io/docs/installation/
kubectl apply -f cert-manager.yaml

If you're still getting the cert-manager HTTP01 challenges failing error.

kubectl get deployments -n cert-manager,
kubectl get deployment cert-manager -n cert-manager -o yaml hostNetwork:false to true :)


kubectl get Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,Challenges --all-namespaces



template:
    metadata:
      annotations:
        prometheus.io/path: /metrics
        prometheus.io/port: "9402"
        prometheus.io/scrape: "true"
      creationTimestamp: null
      labels:
        app: cert-manager
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: cert-manager
        app.kubernetes.io/name: cert-manager
        app.kubernetes.io/version: v1.18.2
    spec:
      hostNetwork: true
      containers:
