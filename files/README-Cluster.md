След като успешно си пуснал kubeadm init --config=configNEW.yaml на master01, следва да копираш PKI сертификатите на другите мастър възли (master02 и master03), за да могат да се включат като control plane nodes.
Стъпки за копиране на сертификатите:

    На master01 създай архив с PKI директорията:

sudo tar czf /tmp/pki-backup.tar.gz -C /etc/kubernetes pki

    Копирай архива към другите мастър възли (пример с scp):

scp /tmp/pki-backup.tar.gz user@192.168.88.129:/tmp/
scp /tmp/pki-backup.tar.gz user@192.168.88.130:/tmp/

(замени user с твоя потребител за SSH достъп до другите сървъри)

    На всеки от другите мастър възли (master02 и master03) разархивирай:

sudo tar xzf /tmp/pki-backup.tar.gz -C /etc/kubernetes/


Грешката The connection to the server localhost:8080 was refused означава, че командата kubectl не може да се свърже към Kubernetes API сървъра от машината, на която я изпълняваш (node01).
Причини и решения:

    kubectl се пуска на нода, но kubeconfig липсва или не е настроен правилно

kubectl използва конфигурационен файл (обикновено ~/.kube/config), за да се свърже към API сървъра. Ако липсва или няма достъп до този файл, командата няма към кой да се свърже.
Какво да направиш:
Вариант 1: Изпълни kubectl на master node

На master01 (или друг master, където си инсталирал kubeadm), командата ще работи, защото там kubeconfig е правилно настроен.
Вариант 2: Копирай kubeconfig на node01

Ако искаш да пускаш kubectl от node01, трябва да имаш конфигурационния файл.

От master01 (примерно):

scp /etc/kubernetes/admin.conf user@node01:/home/user/.kube/config

(сменяш user и пътя според нуждите)

След това на node01:

export KUBECONFIG=/home/user/.kube/config
kubectl get pods -n kube-system

Вариант 3: Използвай kubectl през SSH тунел или пряко от master

Най-често проверките и администрирането се правят от master.

Как да го оправиш:

    Влез на master node (контролер нода), примерно на 192.168.88.128 или където ти е контролерът.

    Копирай файла /etc/kubernetes/admin.conf от master към node01:

scp /etc/kubernetes/admin.conf user@node01:/root/.kube/config

Забележка: Понеже /root/.kube директорията вероятно не съществува, първо създай я на node01:

ssh user@node01
mkdir -p /root/.kube
exit

След това копирай:

scp /etc/kubernetes/admin.conf user@node01:/root/.kube/config

    След като файлът е там, влез на node01 като root и пробвай пак:

kubectl get nodes



Сега, ако искаш да ползваш този kubeconfig на node01 (или друг node), трябва да:

    На master-а пак направиш архив, но само на файла admin.conf, без пътища, примерно:

sudo tar czf /tmp/kubeconfig.tar.gz -C /root/.kube admin.conf

    Копираш архива на node01, примерно:

scp /tmp/kubeconfig.tar.gz user@node01:/tmp/

    На node01 го разархивираш директно в /root/.kube/ (ако тази директория я няма, създай я):

sudo mkdir -p /root/.kube
sudo tar xzf /tmp/kubeconfig.tar.gz -C /root/.kube/
sudo mv /root/.kube/admin.conf /root/.kube/config

Важно: kubeconfig обикновено е само за master или за управление на кластера от външен клиент. На worker нодовете обикновено няма нужда от този файл (освен ако не искаш да управляваш кластера директно от там).

Имаш ли нужда да работиш с kubectl/k9s директно от node01? Ако да — горните стъпки ще ти помогнат да подготвиш kubeconfig там. Ако искаш, помагам с копирането и настройките.

Грешката „tar: admin.conf: Cannot stat: No such file or directory“ означава, че в /root/.kube/ няма файл с име admin.conf.

Провери първо какви файлове има в /root/.kube/:

ls -l /root/.kube/

Вероятно имаш файл config, а не admin.conf, защото ти го преименува преди малко.

Ако искаш да създадеш архив с kubeconfig за копиране, можеш да направиш:

sudo tar czf /tmp/kubeconfig.tar.gz -C /root/.kube config

След това на другия хост (node01) разархивираш config в /root/.kube/ директория.

Кратко:

    В master-а имаш /root/.kube/config, а не admin.conf.

    Архивирай /root/.kube/config вместо admin.conf:

sudo tar czf /tmp/kubeconfig.tar.gz -C /root/.kube config

Ако искаш, кажи ми какво искаш да постигнеш и ще ти помогна да го направиш правилно.

От това, което виждам, /root/.kube/config е налице и с коректни права (-rw-------), което е правилно.

Какво точно разбираш под "прецаках си конфига"? Какво не работи или каква грешка получаваш, когато ползваш kubectl или k9s?

Ако искаш, мога да ти помогна да възстановиш конфигурацията или да проверим дали kubeconfig е валиден и с правилните настройки.

Ето няколко проверки, които можеш да направиш:

    Пробвай:

kubectl get nodes
