![image](https://github.com/user-attachments/assets/e2cffa29-7963-4cea-8fbb-a4be1b9a6e77)

Кластер Kubernetes cluster состоит из набор машин, так называемые узлы, которые запускают контейнеризированные приложения. Кластер имеет как минимум один рабочий узел.

В рабочих узлах размещены поды, являющиеся компонентами приложения. Управляющий слой (control plane) управляет рабочими узлами и подами в кластере. В промышленных средах управляющий слой обычно запускается на нескольких компьютерах, а кластер, как правило, развёртывается на нескольких узлах, гарантируя отказоустойчивость и высокую надёжность.

![image](https://github.com/user-attachments/assets/5e389982-9369-4ba9-9002-bde1366c9456)

# Control Plane Components (Компоненты управляющего слоя)

Компоненты управляющего слоя (control plane) отвечают за основные операции кластера (например, планирование), а также обрабатывают события кластера (например, запускают новый под, когда поле replicas развертывания не соответствует требуемому количеству реплик).

Компоненты управляющего слоя могут быть запущены на любой машине в кластере. Однако для простоты сценарии настройки обычно запускают все компоненты управляющего слоя на одном компьютере и в то же время не позволяют запускать пользовательские контейнеры на этом компьютере. 

## kube-apiserver

Сервер API — компонент Kubernetes панели управления, который представляет API Kubernetes. API-сервер — это клиентская часть панели управления Kubernetes. 

Основной реализацией API-сервера Kubernetes является kube-apiserver. kube-apiserver предназначен для горизонтального масштабирования, то есть развёртывание на несколько экземпляров.

## etcd
Распределённое и высоконадёжное хранилище данных в формате "ключ-значение", которое используется как основное хранилище всех данных кластера в Kubernetes.

## kube-scheduler
Компонент плоскости управления, который отслеживает созданные поды без привязанного узла и выбирает узел, на котором они должны работать.

## kube-controller-manager
Компонент Control Plane запускает процессы контроллера.

Вполне логично, что каждый контроллер в свою очередь представляет собой отдельный процесс, и для упрощения все такие процессы скомпилированы в один двоичный файл и выполняются в одном процессе.

Эти контроллеры включают:

- Контроллер узла (Node Controller): уведомляет и реагирует на сбои узла.
- Контроллер репликации (Replication Controller): поддерживает правильное количество подов для каждого объекта контроллера репликации в системе.
- Контроллер конечных точек (Endpoints Controller): заполняет объект конечных точек (Endpoints), то есть связывает сервисы (Services) и поды (Pods).
- Контроллеры учетных записей и токенов (Account & Token Controllers): создают стандартные учетные записи и токены доступа API для новых пространств имен.

## cloud-controller-manager
cloud-controller-manager запускает контроллеры, которые взаимодействуют с основными облачными провайдерами.

cloud-controller-manager запускает только циклы контроллера, относящиеся к облачному провайдеру. Вам нужно отключить эти циклы контроллера в kube-controller-manager. Вы можете отключить циклы контроллера, установив флаг --cloud-provider со значением external при запуске kube-controller-manager.

С помощью cloud-controller-manager код как облачных провайдеров, так и самого Kubernetes может разрабатываться независимо друг от друга. В предыдущих версиях код ядра Kubernetes зависел от кода, предназначенного для функциональности облачных провайдеров. В будущих выпусках код, специфичный для облачных провайдеров, должен поддерживаться самим облачным провайдером и компоноваться с cloud-controller-manager во время запуска Kubernetes.

Следующие контроллеры зависят от облачных провайдеров:

- Контроллер узла (Node Controller): проверяет облачный провайдер, чтобы определить, был ли удален узел в облаке после того, как он перестал работать
- Контроллер маршрутов (Route Controller): настраивает маршруты в основной инфраструктуре облака
- Контроллер сервисов (Service Controller): создаёт, обновляет и удаляет балансировщики нагрузки облачного провайдера.
- Контроллер тома (Volume Controller): создаёт, присоединяет и монтирует тома, а также взаимодействует с облачным провайдером для оркестрации томов.

# Node Components (Компоненты узла)
Компоненты узла работают на каждом узле, поддерживая работу подов и среды выполнения Kubernetes.

## kubelet
Агент, работающий на каждом узле в кластере. Он следит за тем, чтобы контейнеры были запущены в поде.

Утилита kubelet принимает набор PodSpecs, и гарантирует работоспособность и исправность определённых в них контейнеров. Агент kubelet не отвечает за контейнеры, не созданные Kubernetes.

## kube-proxy
kube-proxy — сетевой прокси, работающий на каждом узле в кластере, и реализующий часть концепции сервис.

kube-proxy конфигурирует правила сети на узлах. При помощи них разрешаются сетевые подключения к вашими подам изнутри и снаружи кластера.

kube-proxy использует уровень фильтрации пакетов в операционной системы, если он доступен. В противном случае, kube-proxy сам обрабатывает передачу сетевого трафика.

## Среда выполнения контейнера
Иcполняемая среда контейнера — это программа, предназначенная для запуска контейнера в Kubernetes.

Kubernetes поддерживает различные среды для запуска контейнеров: Docker, containerd, CRI-O, и любые реализации Kubernetes CRI (Container Runtime Interface).

---
<details> <summary>Максимально кратко</summary>

Control Plane - это мозг Kubernetes, который управляет всем кластером.

* kube-apiserver: Обеспечивает API для взаимодействия с кластером.
* etcd: Хранит конфигурацию кластера, состояния объектов и др. 
* kube-scheduler: Планирует размещение под'ов на узлах.
* kube-controller-manager: Управляет состоянием кластера, запускает контроллеры (например, для ресурсов, pod'ов, служб). 
* Cloud Controller Manager (CCM): Взаимодействует с облачной платформой для предоставления услуг (например, балансировка нагрузки, автомасштабирование).

Node - это "тело" Kubernetes, где работают приложения.

* kubelet: Управляет под'ами на узле, запускает и следит за их состоянием.
* kube-proxy: Обеспечивает сеть между pod'ами и сервисами.
* Container Runtime: Запускает контейнеры (например, Docker, containerd, CRI-O).

</details>

---

# Объекты Kubernetes
Kubernetes содержит ряд абстракций, которые представляют состояние вашей системы: развернутые контейнеризованные приложения и рабочие нагрузки, связанные с ними сетевые и дисковые ресурсы и другую информацию о том, что делает ваш кластер. Эти абстракции представлены объектами в API Kubernetes.

Основные объекты Kubernetes включают в себя:

- Pod (Контейнер или группа контейнеров, которые работают вместе и совместно используют ресурсы.)
- Service (Логическая абстракция группы идентичных Pod'ов, которые предоставляют стабильный доступ к сервису.)
- Том (Постоянное хранилище, которое можно примонтировать к Pod'ам.)
- Namespace (Логическая группировка ресурсов, которая изолирует их от других ресурсов в кластере.)
Kubernetes также содержит абстракции более высокого уровня, которые опираются на Контроллеры для создания базовых объектов и предоставляют дополнительные функциональные и удобные функции. Они включают:

- Deployment (Стратегическое обновление и управление Pod'ами в течение жизненного цикла приложения.)
- DaemonSet (Набор Pod'ов, которые работают на каждом узле кластера и выполняют определенные задачи.)
- StatefulSet (Набор Pod'ов с устойчивыми идентификаторами, которые сохраняют свое состояние даже после перезапуска или масштабирования.)
- ReplicaSet (Набор идентичных Pod'ов, которые обеспечивают отказоустойчивость и масштабируемость.)
- Job (Набор одноразовых задач, которые выполняются до завершения без создания постоянных ресурсов.)

# Minikube ![b6a28b00-9d24-11e9-8a4a-835c1d172ad9_photo-resizer ru](https://github.com/user-attachments/assets/e9f23993-be47-4624-8131-0d00f49e3acb)

## [Install Tools](https://kubernetes.io/docs/tasks/tools/):

----  
### [Docker](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
<details> <summary><kbd>Ctrl</kbd>+<kbd>C</kbd> and <kbd>Ctrl</kbd> + <kbd>V</kbd></summary>
  
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER && newgrp docker
docker --version
```

</details>

----

### [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
<details> <summary><kbd>Ctrl</kbd>+<kbd>C</kbd> and <kbd>Ctrl</kbd> + <kbd>V</kbd></summary>
  
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl 
chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/kubectl
sudo chmod +x /usr/local/bin/kubectl
kubectl version
``` 
</details>

----

### [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download)

<details> <summary><kbd>Ctrl</kbd>+<kbd>C</kbd> and <kbd>Ctrl</kbd> + <kbd>V</kbd></summary>
  
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version
```

</details>

----

```minikube start (по дефолту docker )``` ```# --vm-driver=docker/virtualbox``` запуск Minikube

```minikube stop ``` остановка кластера minikube

```minikube delete``` удаление кластера

## Dashboard UI <img src="https://skillicons.dev/icons?i=sequelize" />

[Install](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

```kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml```

<details> <summary><kbd>admin-user-service-account.yaml</kbd></summary>

```  
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

</details>

<details> <summary><kbd>admin-user-cluster-role-binding.yaml</kbd></summary>

``` 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

```
</details>

```kubectl apply -f admin-user-service-account.yaml -f admin-user-cluster-role-binding.yaml```

```kubectl -n kubernetes-dashboard create token admin-user``` копируем токен

```kubectl proxy``` на 2ой вкладке

```http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/``` вводим токен

Десктоп-альтернатива [Lens](https://k8slens.dev/download)

# Kubernetes <img src="https://skillicons.dev/icons?i=kubernetes" />

```kubectl set image deployment/kuber kuber-app=bokovets/kuber:0.1``` поменять имедж

```kubectl .... --record ``` записать историю ревизий

```kubectl rollout history deployment kuber``` история ревизий( посмотреть почему деплой был обновлён )

```kubectl rollout undo deployment kuber``` откат на версию назад #========================================# ```kubectl rollout undo deployment kuber --to-revision=1```к конкретной ревизии

curl http://kuber-service.default.svc.cluster.local

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
