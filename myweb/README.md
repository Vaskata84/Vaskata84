<div class="Box-sc-g0xbh4-0 bJMeLZ js-snippet-clipboard-copy-unpositioned" data-hpc="true">
<article class="markdown-body entry-content container-lg">
<p dir="auto"><span style="color: #000000;"><strong><a href="https://supportpc.org/">Kubernetes</a></strong></span></p>
<p dir="auto"><span style="color: #000000;"><strong>Вашия уеб сървър с Nginx, в който да можете да добавяте свои HTML или PHP файлове.</strong></span></p>
<p dir="auto"><span style="color: #000000;"><strong>Как да използвате тези файлове:</strong></span></p>
<p dir="auto"><span style="color: #008000;">kubectl apply -f namespace.yaml</span></p>
<p dir="auto"><span style="color: #008000;">Създайте PersistentVolumeClaim:</span></p>
<p dir="auto"><span style="color: #008000;">kubectl apply -f nfspvc.yaml</span></p>
<p dir="auto"><span style="color: #008000;">Създайте Service:</span></p>
<p dir="auto"><span style="color: #008000;">kubectl apply -f service.yaml</span></p>
<p dir="auto"><span style="color: #008000;">Създайте StatefulSet:</span></p>
<p dir="auto"><span style="color: #008000;">kubectl apply -f statefulset.yaml</span></p>
  <p dir="auto"><span style="color: #008000;">Създайте StatefulSet-Resources:</span></p>
<p dir="auto"><span style="color: #008000;">kubectl apply -f statefulset-resources.yaml</span></p>
<p dir="auto"><span style="color: #008000;">Създайте Ingress:</span></p>
<p dir="auto"><span style="color: #008000;">kubectl apply -f ingress.yaml</span></p>
<p dir="auto"><em><strong>След като тези файлове се приложат, вашето приложение ще бъде достъпно на домейн името, което сте задали в Ingress файловете. Можете да добавите своите HTML или PHP файлове, като ги поставите в монтажния път /usr/share/nginx/html, който е дефиниран във вашия StatefulSet.</strong></em></p>
</article>
</div>
