# Kubernetes

![](https://raw.githubusercontent.com/mstrYoda/kubernetes-kitap/master/kubernetes.png)

"GO" dilinde, Google tarafından geliştirilmiş olan Kubernetes, Cloud Native Computing Foundation tarafından desteklenen bir konteyner kümele aracıdır. Konteyner kümelemede (Container Cluster), mevcut olarak konteyner haline getirilen tüm uygulamalar, otomatik olarak harekete geçirilir (deploy edilir), sayıların arttırılıp, azaltılması işlemler aracılığıyla da yönetilmesi sağlanır. Tüm bu işlemleri sağlayan konteyner kümele sistemi de Kubernetes olarak ifade edilir.
 
## Kubernetes Mimarisi
![](https://d33wubrfki0l68.cloudfront.net/7016517375d10c702489167e704dcb99e570df85/7bb53/images/docs/components-of-kubernetes.png)


Bir kubernetes cluster, bir master node’dan ve bir dizi worker node’dan oluşur. Cluster, API çağrıları yoluyla hem iç hemde dış trafiği controller’lara yönlendirir.

### Master Node
Master node, cluster içerisindeki çeşitli sunucu ve yönetici işlemlerini yürütür. Master node’un bileşenleri arasında kube-apiserver, kube-scheduler ve etcd veritabanı bulunmaktadır.
### Kube-Apiserver
![](https://mstryoda.github.io/kubernetes-kitap/images/api-server.png)

Kube-Apiserver, kubernetescluster mekanizmasının ortasında yer almaktadır. Hem içerideki hemde dışarıdaki tüm istekler bu araç üzerinden gerçekleştirilir. Tüm istekler Kube-Apiserver tarafından kabul edilir ve doğrulanır, ayrıca etcd veritabanına yapılan tek bağlantı Kube-Apiserver tarafından yapılır. Sonuç olarak kubernetes in beynidir.
### Kube-Schedular
Bir pod’un hangi node üzerinde çalışacağına karar verir , kubelet’i tetikler ve ilgili pod ve içindeki konteyner oluşturulur.Kısacası yeni bir pod oluşturulması isteğine karşı API server’ı izler.
### Etcd Veritabanı
 Tüm cluster verisinin saklandığı yerdir. Cluster durumu, ağ iletişimi etcd ağaçlarında key-value olarak depolar.
  
### Worker Node
Worker node ise işlerin yürüdüğü yani containerlarımızın çalıştığı node’lara verilen isimdir. Tüm worker node’lar kubelet ve kube-proxy’i ve container engine’i çalıştırır.
### Kubelet
Node üzerindeki kubernetes ajanıdır. Kubernetes’ın tüm nodelarda bulunan agentıdır. Tüm iletişim bu agent üzerinden gerçekleşir. Kube API SERVER dan gelecek isteklere karşı kube API SERVER izler. İlgili docker servisi konuşarak POD’u ayağa kaldırır .En önemli özelliği ; Sürekli olarak node da çalışan containerların ayakta olup-olmadığını kontrol eder ve sürekliliği sağlar.
### Kube-Proxy
![](https://miro.medium.com/max/1400/1*domcyQgFDDpdZHVMOXWn-g.png)

Kubernetes network’u diyebiliriz.Pod’lara IP adresi proxy sayesinde atanır.Bir pod’un içindeki tüm konteyner’lar bir adet paylaşımlı IP ‘yi kullanır.Kube-proxy aynı zamanda bir servisin altındaki tüm pod’lara load-balance özelliği kazandırır.
### Container Engine
Konteyner yönetimini yapar.İmage’ları ilgili registry üzerinden çeker ve konteyner’ların start ve stop olmalarını sağlar.

### Pod 
![](https://miro.medium.com/max/1754/1*9EJSoqwpp-VpWedeqjwSvQ.png)

Container’ların içinde çalıştığı komponentlerdir. En küçük atanabilir birimdir. Kendilerine has IP’leri vardır.İçlerinde bir veya daha fazla container bulundurabilir. Genelde bir container en istenilenidir, bir pod’a çok fazla container koymak mimari açıdan çok doğru olmayabilir. Pod’ların en büyük avantajı kopyalanabilir olmalarıdır.

