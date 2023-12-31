# k8s-yamls
My some Kubernetes cheatsheets

Oficial Kubernetes cheatsheet: https://kubernetes.io/ru/docs/reference/kubectl/cheatsheet/
## k8s autocomplete

```
sudo apt-get install bash-completion
source <(kubectl completion bash) # настройка автодополнения в текущую сессию bash, предварительно должен быть установлен пакет bash-completion .
echo "source <(kubectl completion bash)" >> ~/.bashrc # добавление автодополнения autocomplete постоянно в командную оболочку bash.
```

## k8s alias

```
alias k=kubectl
complete -F __start_kubectl k
```

# Basic

kubectl version

kubectl apply -f <путь до файла или прямая ссылка на файл>

kubectl delete -f <путь до файла или прямая ссылка на файл>

# Minikube
Запустить кластер с именем
```
minikube start --profile k8s-cluster-1
minikube start --profile k8s-cluster-2
minikube start --profile minikube
```

Удалить кластер с именем
```
minikube delete --profile minikube
```

Статус кластера
minikube status --profile k8s-cluster-1

Узнать ip кластера minikube
```
minikube ip --profile k8s-cluster-1
```

Открыть доступ к конкретному сервису в minikube из хост системы
```
minikube service nodeport-service --url --profile k8s-cluster-1
```

Просмотр и добавление дополнений в minikube
```
minikube addons list
minikube addons enable ingress
```

# Dashboard
https://github.com/bgelov/k8s-yamls/tree/main/web-ui-dashboard

Get token
kubectl -n kubernetes-dashboard create token admin-user

Start dashboard

kubectl proxy
Dashboard link: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

# Config
kubectl config get-clusters

kubectl config use-context k8s-cluster-1
kubectl config use-context k8s-cluster-2
kubectl config use-context minikube

Установить неймспейс
```
kubectl config set-context --current --namespace=my-namespace
```

# Nodes

kubectl get nodes
kubectl get nodes -o wide

# Pods

kubectl get pods --all-namespaces 

Чтобы отслеживать изменения в реальном времени
kubectl get pods --all-namespaces --watch

kubectl run xxx-app --image=bgelov/xxxx --port 8008
kubectl get pods 

Выгрузить конфигурацию в yaml
kubectl get pod bgelov-app -o yaml

kubectl exec -it bgelov-app -- /bin/bash
kubectl exec -it bgelov-app -- /bin/sh

kubectl get pod bgelov-app -o yaml

Переадресация портов на хост систему для отладки
```
kubectl port-forward bgelov-app 8888:8888
```

Удалить под
kubectl delete po bgelov-app

Удалить под в конкретном неймспейсе
kubectl delete -n default pod bgelov-rc-pf5lj

Удалить под с определённой меткой
kubectl delete po -l app=nginx

# Pod Logs

kubectl logs bgelov-app

Если несколько контейнеров в Pod, то указываем конкретный контейнер
kubectl logs bgelov-app --container bgelov-app

При удалении подов логи не сохраняются. Надо делать общекластерное ведение логов.


# Pod labels
Лейблы могут быть присоединены к любому объекту k8s
Посмотреть все метки
kubectl get pods --show-labels

Посмотреть только с меткой app
kubectl get pods -l app

Посмотреть только с меткой app и environment 
kubectl get pods -L app,environment 

Селекторы
kubectl get pods -l environment=dev
kubectl get pods -l environment!=dev
kubectl get pods -l environment=dev,app=nginx

kubectl get pods -l !environment
kubectl get pod -l "environment in (dev)"
kubectl get pods -l "app notin (nginx)"


Пометить pod
kubectl label pods bgelov-app-2 environment=dev

Пометить node
kubectl label node k8s-cluster-1 gpu=false

# Annotate
Например, чтобы знать кто создал под
kubectl annotate po bgelov-app-gpu-false company-name/creator-email="bgelov"

# Namespaces
Пространства имён
kubectl get ns

kubectl create namespace qa

```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

Создадим тестовые поды в разных неймспейсах: bgelov-pod-with-namespaces.yaml

kubectl get po --all-namespaces


Удалить неймспейс вместе с подами внутри
kubectl delete ns qa


# ReplicationController
Является устаревшим, переходим на ReplicaSet

Просмотр ReplicationController
kubectl get rc

Удаление ReplicationController 
kubectl delete rc bgelov-rc

Метка в шаблоне должна совпадать с селектором `app: myapp`

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp
spec:
  replicas: <Replicas>
  selector:
    app: myapp
  template:
    metadata:
      name: myapp
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: <Image>
          ports:
            - containerPort: <Port>
```

# ReplicaSet
Заменяет 

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: xiu-rs-2
  labels:
    app: xiu
spec:
  replicas: 3
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - xiu
          - http-server
  template:
    metadata:
      labels:
        app: xiu
        env: dev
    spec:
      containers:
        - name: xiu-app
          image: bgelov/1687346100-977d03e7f0746077d90baa216bbf61c2:1.0.0
          ports:
            - containerPort: 1935

```   


# Deployment
Деплоймент занимается репликасетами, а репликасеты занимаются конкретными подами

kubectl get deployment

kubectl create deployment xiu-app-depl --image=bgelov/1687346100-977d03e7f0746077d90baa216bbf61c2:1.0.0 --port=8888 --replicas=3

Изменим образ у подов деплоймента. Удалятся со старым образом и создадутся поды с новым образом.
kubectl set image deployment/xiu-app-depl 1687346100-977d03e7f0746077d90baa216bbf61c2=nginx --record
Останется старый репликасет, он станет пустым. Пустой репликасет удалять не нужно.

Посмотреть yaml деплоймента
kubectl get deployment xiu-app-depl -o yaml

Удалить деплоймент в неймспейсе default
kubectl delete -n default deployment nginx-temp


Стратегия RollingUpdate, когда добавляем по одной ноде и после удаляем по одной ноде
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: xiu
  labels:
    app: xiu
spec:
  replicas: 5
  minReadySeconds: 10
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  selector:
    matchLabels:
      app: xiu-app
  template:
    metadata:
      labels:
        app: xiu-app
    spec:
      containers:
      - name: xiu-app
        image: bgelov/1687346100-977d03e7f0746077d90baa216bbf61c2:1.0.0
        ports:
        - containerPort: 1935

```

Стратегия Recreate, когда всё удаляем и всё добавляем
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: xiu
  labels:
    app: xiu
spec:
  replicas: 5
  minReadySeconds: 10
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: xiu-app
  template:
    metadata:
      labels:
        app: xiu-app
    spec:
      containers:
      - name: xiu-app
        image: bgelov/1687346100-977d03e7f0746077d90baa216bbf61c2:1.0.0
        ports:
        - containerPort: 1935

```

# Rollout
Откат происходит к предыдущей репликасет. Имменно поэтому их не стоит удалять, при изменении деплоймента.
Посмотреть историю куда можно откатиться
kubectl rollout history deployment/xiu

Откатиться назад
kubectl rollout undo deployment xiu

Откатиться до ревизии 1
kubectl rollout undo deployment xiu --to-revision=1



# Service
Предоставляет единую точку входа к группе подов, представляющих одно и то же приложение.
Работает на основе селектора.
Есть 4 типа:
- ClusterIP (дефолтный, если не указывать). Только в пределах кластера, из вне и из интернета нельзя достигнуть.
- NodePort. Прокидываем какой-либо порт из вне.
- LoadBalancer. Только если кубер крутится на клауд провайдере. Почти как NodePort, но ещё появляется load balancer облачного провайдера.
- ExternalName. Когда хотим подключаться к какому-то внешнему сервису из кластера. Например, к внешней базе данных. По сути создаётся cname с кластерного имени на внешний домен. Здесь стоит иметь ввиду, что http и https будут работать неправильно из-за различных доменов в хеадере.


Пример externalname сервиса
```
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  type: ExternalName
  externalName: bgelov.ru
```


пример NodePort:
```
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  # Чтобы трафик поступал на одну и ту же ноду. На все поды на этой ноде. Если на какой-то ноде не будет пода, то будет зависание.
  # externalTrafficPolicy: Local
  # Если хотим, чтобы клиенты попадали на один и тот же под
  sessionAffinity: ClientIP
  selector:
    app: xiu-app
  ports:
  - protocol: TCP
    port: 8888
    targetPort: 1935
    nodePort: 30080 # Можно не указывать 30000 - 32767 и даже лучше не указывать. k8s сам выберет порт
  type: NodePort
```


Пример loadbalancer

Получить сервисы
kubectl get svc

На имя сервиса можно обращаться:
http://<имя сервиса>.<namespace>.svc.cluster.local
cat /etc/resolv.conf


```
apiVersion: v1
kind: Service
metadata:
  name: xiu-service
spec:
  selector:
    app: xiu-app
  ports:
  - protocol: TCP
    port: 8888
    targetPort: 1935
  type: NodePort
```

Headless сервисы, без кластерного ip. По dns имени будут резолвиться все поды.
```
apiVersion: v1
kind: Service
metadata:
  name: xiu-service
spec:
  ClusterIP: None
  selector:
    app: xiu-app
  ports:
  - protocol: TCP
    port: 8888
    targetPort: 1935
  type: ClusterIP
  ```

# Endpoints 

kubectl get endpoints


# Ingress
Работает на 7 уровне, на http
Когда мы создаём сервис loadbalancer, у нас для каждого создаётся свой лоадбалансер. Он работает на 4 уровне, на транспортном.

Добавление ingress сервиса в minikube
```
minikube addons list
minikube addons enable ingress
```

kubectl get ingress


# Events

Проверка событий в кластере
kubectl get events --watch


# Liveness Probe
Проверка живучести контейнера. Периодически происходит проверка и если она не проходит, то под автоматически перезапускается.
Три механизма:
- http get
Обращается по урлу и если там ошибка, сработает проверка
```
    spec:
      containers:
      - name: kuber-app
        image: bakavets/kuber:v1.0-unhealthy
        ports:
        - containerPort: 8000
        livenessProbe:
          httpGet:
            path: /healthcheck
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
```

httpHeaders для httpGet когда нам нужно следить за определённым именем хоста на поде

```
  template:
    metadata:
      labels:
        app: http-server-http-with-host-headers
    spec:
      containers:
      - name: kuber-app
        image: bakavets/kuber:livenessprobe-http-with-host-headers
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            httpHeaders:
            - name: Host
              value: kuber-healthy.example.com
            port: 80
          # initialDelaySeconds: 5
          periodSeconds: 5
```



- tcp
Проверка tcp сокета. Пытается открыть подключение к указанному порту.
```
      containers:
      - name: kuber-app
        image: bakavets/kuber:v1.0
        ports:
        - containerPort: 8000
        livenessProbe:
          tcpSocket:
            port: 8000
          initialDelaySeconds: 15 # Defaults to 0 seconds. Minimum value is 0.
          periodSeconds: 10 # Default to 10 seconds. Minimum value is 1.
          timeoutSeconds: 1 # Defaults to 1 second. Minimum value is 1.
          successThreshold: 1 # Defaults to 1. Must be 1 for liveness and startup Probes. Minimum value is 1.
          failureThreshold: 3 # Defaults to 3. Minimum value is 1.
---
apiVersion: v1
kind: Service
metadata:
  name: kuber-service-tcp
spec:
  selector:
    app: http-server-tcp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
      nodePort: 30002
  type: NodePort
```


- exec
Выполняется каякая-либо exec команда
```
...
      containers:
      - name: ubuntu
        image: ubuntu
        args:
        - /bin/sh
        - -c
        - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
        livenessProbe:
          exec:
            command:
            - cat
            - /tmp/healthy
          # Следующие параметры задаются по умолчанию, их необязательно указывать
          # Количество секунд от старта контейнера до начала liveness probe
          initialDelaySeconds: 5 # Defaults to 0 seconds. Minimum value is 0.
          # Длительность времени между пробами
          periodSeconds: 5 # Default to 10 seconds. Minimum value is 1.
          # Количество секунд ожидания проблы
          timeoutSeconds: 1 # Defaults to 1 second. Minimum value is 1.
          # Минимальное количество успешных проверок после падения
          successThreshold: 1 # Defaults to 1. Must be 1 for liveness and startup Probes. Minimum value is 1.
          # Количество плохих проверок, перед тем как считать проверку провалившейся
          failureThreshold: 3 # Defaults to 3. Minimum value is 1.
```


# Readiness Probes
Аналогично Liveness Probes, все те же методы, но работает для другого.
Если Readiness Probes не будут проходить, то под не будет регистрироваться к сервису. И следовательно на этот под не будет поступать клиентский трафик.
Readiness Probes часто используют совместно с Liveness Probes.

```
    spec:
      containers:
      - name: kuber-app
        image: bakavets/kuber:v1.0-unhealthy
        ports:
        - containerPort: 8000
        readinessProbe:
          httpGet:
            path: /healthcheck
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /healthcheck
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
```


# Startup Probe
Иногда приложению может потребоваться больше времени для инициализации.
Startup Probe устанавливается большее время для старта контейнера.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuber-http-allprobes
  labels:
    app: kuber
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kuber-http-allprobes
  template:
    metadata:
      labels:
        app: kuber-http-allprobes
    spec:
      containers:
      - name: kuber-app
        image: bakavets/kuber:v1.0
        ports:
        - containerPort: 8000
        startupProbe:
          exec:
            command:
            - cat
            - /server-test.py
          initialDelaySeconds: 10
          failureThreshold: 30 # 30 * 10 = 300 + 10 = 310 sec
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 8000
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          exec:
            command:
            - cat
            - /server-test.py
          failureThreshold: 1
          periodSeconds: 10
  ```


# Переопределение CMD и ENTRYPOINT Docker иструкций

Для переопределения CMD инструкций, мы их указываем в args аттрибуте контейнера

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuber-args
  labels:
    app: kuber
spec:
  replicas: 1
  selector:
    matchLabels:
      app: http-server-args
  template:
    metadata:
      labels:
        app: http-server-args
    spec:
      containers:
      - name: kuber-app
        image: bakavets/kuber:v1.0-args
        args: 
        - "3"
        - "2"
        - text-temp
        ports:
        - containerPort: 8000
```

Для переопределения ENTRYPOINT мы пишем инструкцию command
```
  template:
    metadata:
      labels:
        app: http-server-args
    spec:
      containers:
      - name: kuber-app
        image: bakavets/kuber:v1.0-args
        command: ["xiu", "-c", "/etc/xiu/config_rtmp.toml"]
        # при указании в столбик числа необходимо брать в двойные кавычки
        args: 
        - "3"
        - "2"
        - text-temp
        ports:
        - containerPort: 8000
```



# DaemonSet
Запускает какой-либо один под на всех нодах или нодах с определённым лейблом.

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kuber-daemonset
  labels:
    app: kuber-daemonset
spec:
  selector:
    matchLabels:
      app: kuber-daemon
  template:
    metadata:
      labels:
        app: kuber-daemon
    spec:
      nodeSelector:
        gpu: "true"
      containers:
      - name: kuber-app
        image: bakavets/kuber
        ports:
        - containerPort: 8000
```

# Volumes
Если мы скачиваем что-то во врайтабл слой контейнера, это медленнее и занимает больше места на диске. В кубернетес можно примонтировать к контейнерам emptyDir, которая будет сохранять файлы на диск ноды и этот волюм будет жить до тех пор, пока живы поды. Если под закрашится и перезапустится, то эмтидир останется. Если под пересоздастся, то эмтидир удалится.
Если мы монтируем эмтидир в каталог, в котором уже существуют данные, то эти данные скроются. Чтобы данные не скрывались, надо монтировать в папку используя параметр subPath.
Пример с эмтидиром:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuber
  labels:
    app: kuber
spec:
  replicas: 3
  selector:
    matchLabels:
      app: http-server
  template:
    metadata:
      labels:
        app: http-server
    spec:
      containers:
      - name: kuber-app-1
        image: bakavets/kuber
        ports:
        - containerPort: 8000
        volumeMounts:
        - mountPath: /cache-1
          name: cache-volume
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html/data
        # - mountPath: /cache-2
          name: cache-volume
          subPath: data
      volumes:
      - name: cache-volume
        emptyDir: {}
```

Данные в эмтидир можно так же сохранять в памяти. Для этого надо прописать параметр medium: Memory:
```
  volumes:
  - name: shared-data
    emptyDir: # {}
       medium: Memory
```


- hostPath. Этот тип не рекомендует использовать сам кубернетес, так как происходит монтирование директории хоста. А если вдруг его используешь, то надо ставить ридонли.
```

apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: bakavets/kuber
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: Directory
  ```






persistentVolumeClaim - заявка на ресурсы
persistentVolume - ресурс
и в самом ресурсе указываем claimName

```
# https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-v1/
apiVersion: v1
kind: PersistentVolume
metadata:
  name: aws-pv-kuber
  labels:
    type: aws-pv-kuber
spec:
  capacity:
    storage: 3Gi
  accessModes: # https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain # https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-v1/#PersistentVolumeSpec # https://kubernetes.io/docs/concepts/storage/persistent-volumes/#recycle
  storageClassName: "" # Empty value means that this volume does not belong to any StorageClass. https://kubernetes.io/docs/concepts/storage/persistent-volumes/#class-1
  awsElasticBlockStore:
    volumeID: "vol-02a71cfd076eac916"
    fsType: ext4
```

```
# https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-v1/
apiVersion: v1
kind: PersistentVolume
metadata:
  name: aws-pv-kuber
  labels:
    type: aws-pv-kuber
spec:
  capacity:
    storage: 3Gi
  accessModes: # https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain # https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-v1/#PersistentVolumeSpec # https://kubernetes.io/docs/concepts/storage/persistent-volumes/#recycle
  storageClassName: "" # Empty value means that this volume does not belong to any StorageClass. https://kubernetes.io/docs/concepts/storage/persistent-volumes/#class-1
  awsElasticBlockStore:
    volumeID: "vol-02a71cfd076eac916"
    fsType: ext4
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuber
  labels:
    app: kuber
spec:
  replicas: 1
  selector:
    matchLabels:
      app: http-server
  template:
    metadata:
      labels:
        app: http-server
    spec:
      containers:
      - name: kuber-app
        image: bakavets/kuber
        ports:
        - containerPort: 8000
        volumeMounts:
        - mountPath: /cache
          name: cache-volume
      volumes:
      - name: cache-volume
        persistentVolumeClaim:
          claimName: aws-pvc-kuber
```





С помощью storageclass можно настроить автоматическсое создание persistentvolume
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-csi-gp3
provisioner: ebs.csi.aws.com
allowVolumeExpansion: true
parameters:
  type: gp3
  fsType: ext4
```





# ConfigMap
Для хранения несекретных данных в формате ключ-значение. Поды могут использовать их как переменные окружения, аргументы командной строки или файлы конфигурации волюмов.
Например, нам надо указать хост базы данных.
Конфиг файлы для приложений.
Если конфиг файлы большие, то обычно используют волюмы.


Пример конфигмапа:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-cm
data:
  # property-like keys; each key maps to a simple value
  interval: "5"
  count: "3"
  # file-like keys
  properties: |
    Hello from World!
    This is demo config!
    As an example.
  config.ini: "This is demo config!"
```

Пример деплоя с конфигмапом:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuber-2
  labels:
    app: kuber-2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: http-server-2
  template:
    metadata:
      labels:
        app: http-server-2
    spec:
      containers:
      - name: kuber-app
        image: bakavets/kuber:v1.0-args
        args: ["$(INTERVAL)","$(COUNT)","$(TEXT_ARG)"]
        ports:
        - containerPort: 8000
        env:
          - name: INTERVAL
            valueFrom:
              configMapKeyRef:
                name: demo-cm
                key: interval
          - name: COUNT
            valueFrom:
              configMapKeyRef:
                name: demo-cm
                key: count
          - name: TEXT_ARG
            valueFrom:
              configMapKeyRef:
                name: demo-cm
                key: properties
        volumeMounts:
        - name: config
          mountPath: "/config"
          readOnly: true
      volumes:
        # You set volumes at the Pod level, then mount them into containers inside that Pod
        - name: config
          configMap:
            # Provide the name of the ConfigMap you want to mount.
            name: demo-cm
            # An array of keys from the ConfigMap to create as files
            items:
            - key: "properties"
              path: "properties"
            - key: "config.ini"
              path: "config.ini"
```

Когдв мы мапим конфиг через volumeMounts, у нас скроются все остальные файлы в каталоге.
Чтобы не скрылись, можно использовать subPath:
```
          volumeMounts:
            - name: nginx-conf
              mountPath: /etc/nginx/conf.d/nginx.conf
              subPath: nginx.conf
              readOnly: true
```

Ниже полный пример

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf-file
  labels:
    app: nginx
data:
  nginx.conf: |
    server {
      listen 80;
      access_log /var/log/nginx/reverse-access.log;
      error_log /var/log/nginx/reverse-error.log;
      location / {
            proxy_pass https://github.com/bakavets;
      }
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-proxy
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
          - containerPort: 80
          volumeMounts:
            - name: nginx-conf
              mountPath: /etc/nginx/conf.d/nginx.conf
              subPath: nginx.conf
              readOnly: true
      volumes:
        - name: nginx-conf
          configMap:
            name: nginx-conf-file
```


Переменные окружения можно объявлять через блок env
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuber-1
  labels:
    app: kuber-1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: http-server-1
  template:
    metadata:
      labels:
        app: http-server-1
    spec:
      containers:
      - name: kuber-app
        image: bakavets/kuber:v1.0
        ports:
        - containerPort: 8000
        env:
        - name: HELLO
          value: "Hello"
        - name: WORLD
          value: "World"
        - name: ENV_HELLO_WORLD
          value: "$(HELLO)_$(WORLD) from Pod"

```


Create a Kubernetes Configmap with custom nginx.conf using kubectl:

kubectl create configmap nginx-config --from-file=nginx.conf

Create ConfigMaps from literal values
You can use kubectl create configmap with the --from-literal argument to define a literal value from the command line:

kubectl create configmap config --from-literal=interval=7 --from-literal=count=3 --from-literal=config.ini="Hello from ConfigMap"

Create ConfigMaps from folder:
kubectl create configmap my-config --from-file=configs/

Use the option --from-env-file to create a ConfigMap from an env-file, for example:
kubectl create configmap config-env-file --from-env-file=env-file.properties


