![image](https://github.com/user-attachments/assets/e2cffa29-7963-4cea-8fbb-a4be1b9a6e77)

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

```
minikube start --vm-driver=docker/virtualbox (по дефолту docker ) # запуск Minikube
```
```
minikube start --profile k8s-cluster-1 # создание локального миникуб кластера
```
```
minikube stop - остановка кластера minikube
```

# Kubernetes <img src="https://skillicons.dev/icons?i=kubernetes" />

```
/home/wireflex/.kube/config # тут находится конфиг-файл kubectl( clusters,contexts,users )
```
```
kubectl config set-cluster my-external-cluster --server=https://k8s.test.com:9443 --certificate-authority=path_to_the/cafile # добавление своего кластера
```
```
kubectl config set-credentials temp --username=temp --password=superroot # новый юзер с логин/паролем
```
```
kubectl config set-credentials temp --token=supertoken # переопределение аутентификации пользователя ( токен вместо пароля ) 
```
```
kubectl config set-context some-context --cluster=my-external-cluster --user=temp --namespace=some-namespace # создание контекста 
```
```
kubectl get pods --all-namespaces # просмотр всех системных подов ( coredns,etcd,apiserver,controller-manager,proxy,scheduler,storage-provisioner )
```
