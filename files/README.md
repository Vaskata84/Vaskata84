<h3><strong>Installing and configuring Etcd on all 3 Master Nodes</strong></h3>

<h4>Step 1 - Download and move etcd files and certs to their respective places</h4>

<pre>
<code class="language-bash">sudo mkdir /etc/etcd /var/lib/etcd

sudo mv ~/ca.pem ~/kubernetes.pem ~/kubernetes-key.pem /etc/etcd

wget https://github.com/Vaskata84/Vaskata84/blob/master/files/etcd-v3.4.13-linux-amd64.tar.gz

tar xvzf etcd-v3.4.13-linux-amd64.tar.gz

sudo mv etcd-v3.4.13-linux-amd64/etcd* /usr/local/bin/
</code></pre>
