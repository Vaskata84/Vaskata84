<p>Как да използвате тези файлове:</p>
<pre><code>Създайте <span class="hljs-string">Namespace:</span>

bash
</code></pre><p>kubectl apply -f namespace.yaml</p>
<p>Създайте PersistentVolumeClaim:</p>
<p>bash</p>
<p>kubectl apply -f nfspvc.yaml</p>
<p>Създайте Service:</p>
<p>bash</p>
<p>kubectl apply -f service.yaml</p>
<p>Създайте StatefulSet:</p>
<p>bash</p>
<p>kubectl apply -f statefulset.yaml</p>
<p>Създайте Ingress:</p>
<p>bash</p>
<pre><code>kubectl apply <span class="hljs-_">-f</span> ingress.yaml
</code></pre><p>След като тези файлове се приложат, вашето приложение ще бъде достъпно на домейн името, което сте задали в Ingress файловете. Можете да добавите своите HTML или PHP файлове, като ги поставите в монтажния път /usr/share/nginx/html, който е дефиниран във вашия StatefulSet.</p>

