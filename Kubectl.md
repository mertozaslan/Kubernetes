# Kubectl
Kubernetes kubectl adında bir CLI içerir kubectl bizleri shell üzerinden çalıştırıp kubernetis api serverı ile iletişim kurarak ona komutlar gönderebileceğimiz kubernetes projesinin resmi CLI aracıdır.
![](https://i0.wp.com/osmansecer.net/wp-content/uploads/2021/07/kubectl-1.jpg?fit=1220%2C577&ssl=1)

### Kubectl kullanımı
Komut Ekranına
```kubectl --help```
Yazıp kullanabileceğimiz komutları görebiliriz.
Kullanım formatı ```kubectl [flags] [options]``` şeklinde olup. İhtiyaç duyulan komutlar hakkında daha detaylı bilgi için ```kubectl [command] --help``` komutu kullanılabilir.

Benzer şekilde herhangi bir objenin ne olduğunu ne işe yaradığını ve buna benzer bilgiler almak için ```explain``` komutu kullanılabilir.

Belirli bir obje hakkında bilgi almak istiyorsak ```describe``` komunutunu kullanabiliriz.

## Imperative ve Declarative
Kubernetes işlemleri imperative ve declarative olarak yapmayı sağlar.
* Imperative yöntemde yapılacak olan işlem komut yorumlayıcısına yazılarak yapılır.

```kubectl run nginx --image=nginx --port=80``` komutu Imperative yönteme bir örnektir. Eğer ki oluşturulan objenin içerisinde bir değişiklik yapılmak istenilirse bu objesinin silinip yeniden oluşturulabilir veya oluşturulmuş bir objenin üzerinde direkt olarak değişiklik yapmak istediğimizde ```kubectl edit``` komutunu kullanırız ve bu komut bizlere seçtiğimiz objeyi yaml dosyası formatında ile açarak varsayılan text editörü ile değişiklik yapmamıza olanak tanır.Özel olarak yukarıda bahsedilen örnek için ```kubectl edit pods nginx``` komutu kullanılır.

* Declarative yöntemde ise yapılacak işlemlerin bir veya birden fazla script içerisinde tanımlanıp yürütülen işlemlerdir. Genellikle yapılacak olan işlem yaml formatında oluşturulur.
Declarative yöntemde Yaml dosyaları 4 ana bölmeden oluşur.
* apiVersion: Burada oluşturmak istediğimiz objenin Kubernetes API versiyonu yazılır.
* kind: Oluşturulmak istenen objenin türü yazılır.
* Metadata: Objenin adını ve çeşitli Labelleri tanımlayabiliriz.
* Spec: Oluşturmak istediğimiz objenin özelliklerini belirtiriz.
 
Declarative yöntem için örneğin nginx.yaml adlı ve içeriği şöyle olan ;
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
      - containerPort: 80
```
bir yaml dosyası oluşturalım.

Ardından ```kubectl apply -f nginx.yaml``` komutu ile pod'u oluştururuz. Yukarıda imparative yöntem ile oluşturulan obje ile tamamen aynıdır fakat declarative yöntem ile objede yapmak istediğimiz değişiklikleri önce yaml dosyası üzerinden yapıp ardından ```kubectl apply``` komutunu girebiliriz bu bize oldukça kolaylık sağlar.

### Multi-Container Pods

Bir pod oluşturulurken genelde oluşturulan pod içerisinden sadece bir uygulama tek bir container halinde olması gerekir fakat bu durum eğer ki kullanılacak uygulamalar birbirine bağımlı ise örneğin ikinci uygulamamız bir log uygulaması ise bu tarz durumlarda tek bir pod içerisinde iki uygulama oluşturmak bizlere kolaylık sağlayabilir. Bu tarz oluşturulan podlara Multi-Container pod denir.

### Init Container
![](https://miro.medium.com/max/1400/1*9erV-cjHBzdKA0Mx5jH0PA.png)

Init containerları da aynı normal containerlar gibidir. En önemli farkı, init containerı başarıyla bitmeden bir sonraki ( bizim ana istediğimiz ) containerlar başlayamaz. Eğer init containerı hata alırsa, hata gidene kadar pod onu restart eder. InitContainerlar genellikle diğer containerlar çalışmaya başlamadan önce pod içerisinde yapılması gereken işleri yapmak veya uygulama containerının başlamasını bekletmek gibi ihtiyaçları karşılamak için kullanılabilirler.

### Deployment
![](https://www.optdcom.net/levitate/wp-content/uploads/2021/07/Rolling-Updates-and-Rollbacks-in-Kubernetes.png)

Uygulamarınızı Kubernetes de deploy ve update etmek için şuan en meşhur ve tavsiye edilen yöntemdir. Bir deployment'da istenilen durumu tanımlarsınız ve deployment controller mevcut durumu istenilen durum ile karşılaştırıp gerekli aksiyonları alır.
* Deployment Oluşturmak : ```kubectl create deployment <deployment-name> --image=<image-name> -n <namespace>``` formatındadır.
* Deployment Scale : ```kubectl scale deplyoment <deployment-name>``` komutunun ardına yapmak istediğimiz değişikliği girebiliriz.

##### Yaml dosyası ile Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
Burada matchLabels ve labels lar aynı olması gerekir çünkü deployment oluşturduğu ReplicaSet sayesinde buradaki labelları kullanarak podları kontrol eder.

#### ReplicaSet
Bir poddan kaç tane olacağını replicaset ile belirtiyoruz.İstediğiniz anda bir pod’u kesintisiz bir şekilde scale edebilir yada azaltabiliriz.

### Service
Service, podlar ve nodelar arasındaki iletişimi sağlar. Kubernetes içerisindeki trafiği yönetmek için servislere ihtiyaç vardır.

Bir yaml dosyası hazılayarak service oluşturulabilir.

Servisin hangi podlar arasındaki iletişimi sağlayacağı label-selector sayesinde belirtilir. pod.yaml’ında label, service.yaml’ında selector tanımlaması yaparak belirtilir.
##### Service’ler 3 farklı tipte olabilir
* ClusterIp: Service’lerin kendi aralarındaki iletişimi sağlar.
###### ClusterIP Manifest Örneği
![](https://miro.medium.com/max/546/1*zfl7CT0syB8NNpVCgvVU0g.png)

* NodePort: Bir Node’un dışarı ile iletişim kurabileceği port’u sağlayan Servis türüdür.
###### Nodeport Manifest Örneği
![](https://miro.medium.com/max/640/1*udsEr0kKCs0rYVRpKSe1yQ.png)

* Load Balancer: Service’e bir Public IP atanması sağlanarak, Service’in dış networkten gelen istekleri karşılayarak ilgili Node’lar dağıtma hizmetini sağlar.
###### Load Balancer Manifest Örneği
![](https://miro.medium.com/max/638/1*s8cvKTKRJ2SAX20gxbQw0w.png)

### Volume
Kubernetes de Volume kavramı Docker Volume ile benzerdir. Container’ın dosyaları container varolduğu sürece bir dosya içerisinde tutulur. Eğer container silinirse, bu dosyalarda birlikte silinir. Bazı containerlarda bu dosyaların silinmemesi gerekebilir. İşte bu durumda Ephemeral (Geçici) Volume kavramı devreye giriyor. Ephemeral Volume’ler aynı pod içerisindeki tüm containerlara aynı anda bağlanabilir. Diğer bir amaç ise aynı pod içerisindeki birden fazla container’ın ortak kullanabileceği bir alan yaratmaktır.Ephemeral Volume’lerde Pod silinirse tüm veriler kaybolur. Fakat, container silinip tekrar yaratılırsa Pod’a bir şey olmadığı sürece veriler saklanır.

2 tip Ephemeral Volume vardır:
* emptyDir Volume: emptyDir volume ilk olarak bir pod vir node'a atandığında oluşturulur ve bu pod o node'da çalıştığı sürece var olur. Bir pod herhangi bir nedenşe bir node'dan silindiğinde emptyDir içindeki veriler de kalıcı olarak silinir.

##### Yaml dosyası ile emptyDir Volume
```
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```
* hostPath Volume : Bir hostPath volume worker node dosya sisteminden pod'unuza bir dosya veya dizini bağlayabilme imkanı verir. emptyDir’da volume klasörü pod içerisinde yaratılırken, hostPath’te volume klasörü worker node içerisinde yaratılır.

##### Yaml dosyası ile hostPath Volume
```
apiVersion: v1
kind: Pod
metadata:
  name: hostpath
spec:
  containers:
  - name: hostpathcontainer
    image: ozgurozturknet/k8s:blue
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /healthcheck
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    volumeMounts:
    - name: directory-vol
      mountPath: /dir1
    - name: dircreate-vol
      mountPath: /cache
    - name: file-vol
      mountPath: /cache/config.json       
  volumes:
  - name: directory-vol
    hostPath:
      path: /tmp
      type: Directory
  - name: dircreate-vol
    hostPath:
      path: /cache
      type: DirectoryOrCreate
  - name: file-vol
    hostPath:
      path: /cache/config.json
      type: FileOrCreate

```

### Secret 
Temel amacı hassas verilerimizi 3. Kişilerin eline geçmemesini sağlamaktır. Özellikle Database barındıracak bir Pod ayağa kaldırdığımızda secret kullanımı kaçınılmazdır. Hassas bilgileri (DB User, Pass vb.) Environment Variables içerisinde saklayabilsekte, bu yöntem ideal olmayabilir.
Secret object’i sayesinde bu hassas bilgileri uygulamanın object tanımlarını yaptığımız YAML dosyalarından ayırıyoruz ve ayrı olarak yönetebiliyoruz.

##### Yaml dosyası ile Secret
```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
stringData:	
  db_server: db.example.com
  db_username: admin
  db_password: P@ssw0rd!
```
Imperative olarak da ```kubectl create secret generic``` ile secret oluşturabiliriz

### ConfigMap
ConfigMap’ler Secret objectleriyle tamamen aynı mantıkta çalışır.ConfigMap gizli olmayan verileri anahtar/değer eşlenikleri olarak depolamak için kullanılan bir API nesnesidir.
##### Yaml dosyası ile ConfigMap
```
apiVersion: v1
 kind: Pod
 metadata:
   name: webapp
   labels:
       app: webapp
 spec:
     containers:
     - image: nginx
       name: nginx
       ports:
       - containerPort: 80
       command: ["/bin/sh", "-c", "echo $(DATA_STRING) > $(DATA_PATH); sleep 3600"]
       env:
        - name: DATA_STRING
          valueFrom:
             configMapKeyRef:
                 name: webnginx
                 key: STRING
                 optional: true
        - name: DATA_PATH
          valueFrom:
             configMapKeyRef:
                 name: webnginx
                 key: PATH
                 optional: true
```

### Node Affinity
Node affinity kavramsal olarak nodeSelector'a benzer ve nodelara atanan etiketlere göre podunuzun hangi node üstünde schedule edilmeye uygun olduğunu kısıtlamanıza olanak tanır.
Mevcut olarak iki tip node affinity vardır.
* requiredDuringSchedulingIgnoredDuringExecution: Bu affinity tipi ile scheduler’ın pod’u mutlaka bu kurala uyan bir node’a ataması zorunlu kılınır.
* preferredDuringSchedulingIgnoredDuringExecution: Bu affinity tipi ile scheduler’ın kuralı uygulamaya çalışması fakat başarılı olamazsa pod’u başka bir node’a da atayabilmesi sağlanır.
##### Yaml dosyası ile Node Affinity
```
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - antarctica-east1
            - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0

```

### Pod Affinity
Pod affinity podunuzun hangi node üstünde oluşturulmaya uygun olduğunu nodelardaki etkitlere göre değil halihazırda node da çalışmakta olan podlardaki etiketlere göre sınırlamanıza olanak tanır.
   
##### Yaml dosyası ile Pod Affinity
```
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - cache
        topologyKey: kubernetes.io/hostname
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - rest-api
          topologyKey: kubernetes.io/hostname
  containers:
  - name: with-pod-affinity
    image: k8s.gcr.io/pause:2.0
  ```
  
  ### DaemonSet
  DaemonSet tüm nodeların bir pod'un bir kopyasını çalıştırmasını sağlar. Clustera yeni node eklendikçe onlara podlar da eklenir. Clusterdan node kaldırıldığında bu podlar da kaldırılır. Bir DaemonSet'in silinmesi oluşturduğu pod'ları da temizleyecektir.
  
  ![](https://miro.medium.com/max/1252/1*YkLBukilVyDmcDdoK3092A.png)





























   
  
