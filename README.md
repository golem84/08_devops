
## 0. Подготовка

### 0.1. Установка Helm
```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
# install HELM:
$ ./get_helm.sh
# check version HELM:
$ helm version
version.BuildInfo{Version:"v3.21.2", GitCommit:"125963406833fe0525be91f46c8b5b0f22fb9e32", GitTreeState:"clean", GoVersion:"go1.26.4"}
```
### 0.2. Установка kompose
```bash
$ curl -L https://github.com/kubernetes/kompose/releases/download/v1.38.0/kompose-linux-amd64 -o kompose
$ chmod +x kompose
$ sudo mv ./kompose /usr/local/bin/kompose
$ kompose version
1.38.0 (a8f5d1cbd)
```

## 1. Подготовка HELM chart:   
```bash
# convert to helm chart
sysadmin@master:~/labs/08_devops/promgrafana$ kompose convert --chart
WARN Restart policy 'unless-stopped' in service grafana is not supported, convert it to 'always'
WARN Restart policy 'unless-stopped' in service prometheus is not supported, convert it to 'always'
WARN File don't exist or failed to check if the directory is empty: stat :/var/lib/grafana: no such file or directory
WARN File don't exist or failed to check if the directory is empty: stat :/prometheus: no such file or directory
INFO Kubernetes file "docker-compose/templates/blackbox-service.yaml" created
INFO Kubernetes file "docker-compose/templates/grafana-service.yaml" created
INFO Kubernetes file "docker-compose/templates/prometheus-service.yaml" created
INFO Kubernetes file "docker-compose/templates/blackbox-deployment.yaml" created
INFO Kubernetes file "docker-compose/templates/grafana-deployment.yaml" created
INFO Kubernetes file "docker-compose/templates/grafana-data-persistentvolumeclaim.yaml" created
INFO Kubernetes file "docker-compose/templates/grafana-cm0-configmap.yaml" created
INFO Kubernetes file "docker-compose/templates/prometheus-deployment.yaml" created
INFO Kubernetes file "docker-compose/templates/prom-data-persistentvolumeclaim.yaml" created
INFO Kubernetes file "docker-compose/templates/prometheus-cm0-configmap.yaml" created
INFO chart created in "docker-compose/"
# check dir tree: 
sysadmin@master:~/labs/08_devops/promgrafana$ tree  docker-compose
docker-compose
├── Chart.yaml
├── README.md
└── templates
    ├── blackbox-deployment.yaml
    ├── blackbox-service.yaml
    ├── grafana-cm0-configmap.yaml
    ├── grafana-data-persistentvolumeclaim.yaml
    ├── grafana-deployment.yaml
    ├── grafana-service.yaml
    ├── prom-data-persistentvolumeclaim.yaml
    ├── prometheus-cm0-configmap.yaml
    ├── prometheus-deployment.yaml
    └── prometheus-service.yaml

2 directories, 12 files
```  
меняем название чарта, сервис grafana-service.yml  
упаковываем чарт promgra из каталога репозитория  
```bash
sysadmin@master:~/labs/08_devops/promgrafana$ helm package promgra
Successfully packaged chart and saved it to: /home/sysadmin/labs/08_devops/promgrafana/promgra-0.0.1.tgz
```

чтобы в linux сделать сервис доступным из minikube снаружи выполнить:  
`$ minikube service <service-name> -n <namespace>`

```bash
$ kubectl get services
NAME             TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)          AGE
kubernetes       ClusterIP      10.96.0.1      <none>           443/TCP          8h
redis            ClusterIP      10.98.46.109   <none>           6379/TCP         7h51m
service-devops   LoadBalancer   10.96.78.208   192.168.11.133   8000:31406/TCP   7h51m
$ minikube service service-devops -n default
┌───────────┬────────────────┬─────────────┬─────────────────────────────┐
│ NAMESPACE │      NAME      │ TARGET PORT │             URL             │
├───────────┼────────────────┼─────────────┼─────────────────────────────┤
│ default   │ service-devops │ 8000        │ http://192.168.39.110:31406 │
└───────────┴────────────────┴─────────────┴─────────────────────────────┘
🎉  Opening service default/service-devops in default browser...
```
