<h3><strong>Installing and configuring Etcd on all 3 Master Nodes</strong></h3>

<h4>Step 1 - Download and move etcd files and certs to their respective places</h4>

<pre>
<code class="language-bash">sudo mkdir /etc/etcd /var/lib/etcd

sudo mv ~/ca.pem ~/kubernetes.pem ~/kubernetes-key.pem /etc/etcd

wget https://github.com/Vaskata84/Vaskata84/blob/master/files/etcd-v3.4.13-linux-amd64.tar.gz

tar xvzf etcd-v3.4.13-linux-amd64.tar.gz

sudo mv etcd-v3.4.13-linux-amd64/etcd* /usr/local/bin/
</code></pre>

<pre>
<code class="language-bash">sudo nano /etc/systemd/system/etcd.service
</code></pre>

<p>Enter the following config:</p>

<pre>
<code class="language-bash">[Unit]
Description=etcd
Documentation=https://github.com/coreos


[Service]
ExecStart=/usr/local/bin/etcd \
  --name 192.168.1.113 \
  --cert-file=/etc/etcd/kubernetes.pem \
  --key-file=/etc/etcd/kubernetes-key.pem \
  --peer-cert-file=/etc/etcd/kubernetes.pem \
  --peer-key-file=/etc/etcd/kubernetes-key.pem \
  --trusted-ca-file=/etc/etcd/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ca.pem \
  --peer-client-cert-auth \
  --client-cert-auth \
  --initial-advertise-peer-urls https://192.168.1.113:2380 \
  --listen-peer-urls https://192.168.1.113:2380 \
  --listen-client-urls https://192.168.1.113:2379,http://127.0.0.1:2379 \
  --advertise-client-urls https://192.168.1.113:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster 192.168.1.113=https://192.168.1.113:2380,192.168.1.114=https://192.168.1.114:2380,192.168.1.115=https://192.168.1.115:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
</code></pre>

<p>Replace the IP address on all fields except the <code>&mdash;initial-cluster</code> field to match the machine IP.</p>

<h4>Step 3 - Reload the daemon configuration.</h4>

<pre>
<code class="language-bash">sudo systemctl daemon-reload
</code></pre>

<h4>Step 4 - Enable etcd to start at boot time.</h4>

<pre>
<code class="language-bash">sudo systemctl enable etcd
</code></pre>

<h4>Step 5 - Start etcd.</h4>

<pre>
<code class="language-bash">sudo systemctl start etcd
</code></pre>

<p><strong>Repeat the process for all 3 master nodes and then move to step 6.</strong></p>

<h3>Step 6 - Verify that the cluster is up and running.</h3>

<pre>
<code class="language-bash">ETCDCTL_API=3 etcdctl member list
</code></pre>
