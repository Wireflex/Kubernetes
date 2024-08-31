# ServiceAccount
это учетная запись, которая позволяет приложениям напрямую общаться с Kubernetes-API без предоставления учетных данных пользователя , то есть предоставляет identity для процессов, запущенных в ПОДЕ

## Roles
содержат правила, предоставляющие набор permissions ( то есть то, что можно делать с этой ролью - листать поды, получать с них логи ) 

### Role работает внутри намспейса( при создании нужно указать намспейс )

### ClusterRole это объект НЕ-намспейсного уровня, а кластерного уровня ( можно определеять permissions на уровне кластера, там же можно описывать другие объекты НЕнамспейсного уровня - PV, StorageClass итд ) 

### RoleBinding необходим для того, чтобы привязать определенный subject (users, groups к определенным ролям) объект namespace уровня, может ссылаться на роль в пределах намспейс, но если нужно привязать ClusterRole ко всем намспейс - юзаем RoleBinding


<details> <summary>app.yaml</summary>

``` 
---
apiVersion: v1
kind: Namespace
metadata:
  name: example-1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuber-1
  namespace: example-1
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
      serviceAccountName: app-sa
      containers:
        - name: kuber-app
          image: bakavets/kuber:v1.0
          ports:
            - containerPort: 8000
---
apiVersion: v1
kind: Service
metadata:
  name: kuber-service-1
  namespace: example-1
spec:
  selector:
    app: http-server-1
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
```
</details>

если не указать ServiceAccount явно - приатачится дефолтный

![image](https://github.com/user-attachments/assets/1b18462a-1431-4b4d-9702-5566d3594bfb)

![image](https://github.com/user-attachments/assets/69e5f543-d66e-44b5-b9b8-3429d5d86703)

В секретах к этому ServiceAccount относится определенный набор credentials

![image](https://github.com/user-attachments/assets/889f958d-cc9d-4fee-97f5-f3ac04c1500f)

за счёт токена кубер-API будет понимать какой identity делает к нему request

![image](https://github.com/user-attachments/assets/d4240e92-bc51-4ee9-af86-a6c5e0515615)

Однако токен внутри ПОДА :point_down: отличается от ТОКЕНА в секрете :point_up:

![image](https://github.com/user-attachments/assets/0090c3dc-7ba6-49ba-8da8-134780939755)

Сертификат и namespace совпадают

![image](https://github.com/user-attachments/assets/dad744da-261b-447d-a98a-9c4fa105c131)

```
# Для общение с кубер-API
APISERVER=https://kubernetes.default.svc

# путь, где лежат данные (токен, namespace сертификат )
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount

# Read this Pod's namespace
NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)

# Read the ServiceAccount bearer token
TOKEN=$(cat ${SERVICEACCOUNT}/token)

# Reference the internal certificate authority (CA)
CACERT=${SERVICEACCOUNT}/ca.crt

# запрос к кубер-API, используя токен и сертификат
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api
```
Вставляем всё прямо в ПОД, ошибки нет - всё работает

![image](https://github.com/user-attachments/assets/cee910da-8920-4f63-9774-1264223a5504)

Если задать левое значение переменной, то будет ошибка

![image](https://github.com/user-attachments/assets/96f126fb-fdf0-4f57-b81e-a93e8e53beee)

Допустим, у нас есть приложение(контроллер) которое будет скейлить деплоймент (вверх/вниз) на основании метрик, для того, чтобы процессы могли это делать, им нужны определенные credentials

с помощью определенного выше :point_up: сертификата(CACERT), токена(TOKEN) и api-сервера(APISERVER) делаем запрос по пути ( /apis/apps/v1/namespaces/${NAMESPACE} ) для деплоймента (deployments), то есть хотим листануть все деплойменты, которые существуют в namespace 'example-1'
```
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/apis/apps/v1/namespaces/${NAMESPACE}/deployments
```
