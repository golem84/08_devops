
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
деплоим нагрузку из helm chart:  
```bash
andrew@andrew-xubuntulp:~/projects/08_devops/promgrafana$ helm install promgra promgra-0.0.1.tgz
NAME: promgra
LAST DEPLOYED: Fri Jun 26 14:07:45 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
проверяем развернутую нагрузку:  
```bash
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
blackbox-d4d587b7b-vxxkr      1/1     Running   0          17m
grafana-9c68799cd-xtnw5       1/1     Running   0          23m
prometheus-5d6f5c49c5-vgdl9   1/1     Running   0          23m
```

Добавляем переменные в helm chart:  
```yaml
EXTERNAL_IP: 192.168.11.133
EXTERNAL_PORT: 3113
GF_ADMIN_PASSWORD: HiGrafana
```
helm lint:  
```bash
andrew@andrew-xubuntulp:~/projects/08_devops/promgrafana/promgra$ helm lint
==> Linting .
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```
проверяем что helm видит переменные:  
```bash
andrew@andrew-xubuntulp:~/projects/08_devops/promgrafana$ helm show values ./promgra/
EXTERNAL_IP: 192.168.11.133
EXTERNAL_PORT: 3113
GF_ADMIN_PASSWORD: HiGrafana
```
и корректно подставляет их в chart:  
```bash
andrew@andrew-xubuntulp:~/projects/08_devops/promgrafana$ helm template promgra ./promgra/
# grafana-service.yaml only
---
# Source: promgra/templates/grafana-service.yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert --chart
    kompose.version: 1.38.0 (a8f5d1cbd)
  labels:
    io.kompose.service: grafana
  name: grafana
spec:
  type: LoadBalancer
  ports:
    - port: 3113
      targetPort: 3000
  externalIPs:
    - 192.168.11.133
  selector:
    io.kompose.service: grafana
```
изменяем релиз, уточняя порт в в переменной EXTERNAL_PORT:  
```bash
andrew@andrew-xubuntulp:~/projects/08_devops/promgrafana/promgra$ helm upgrade promgra ./ --set EXTERNAL_PORT=3489
Release "promgra" has been upgraded. Happy Helming!
NAME: promgra
LAST DEPLOYED: Fri Jun 26 14:44:52 2026
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
```
проверяем, что изменения применились:  
```bash
andrew@andrew-xubuntulp:~/projects/08_devops/promgrafana/promgra$ kubectl get services -n default
NAME             TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)          AGE
blackbox         ClusterIP      10.101.184.212   <none>           9115/TCP         39m
grafana          LoadBalancer   10.101.226.156   192.168.11.133   3489:31479/TCP   39m
kubernetes       ClusterIP      10.96.0.1        <none>           443/TCP          13h
prometheus       ClusterIP      10.106.191.2     <none>           9090/TCP         39m
redis            ClusterIP      10.98.46.109     <none>           6379/TCP         13h
service-devops   LoadBalancer   10.96.78.208     192.168.11.133   8000:31406/TCP   13h
```
check exported variables:  
```bash
andrew@andrew-xubuntulp:~$ helm get values promgra
USER-SUPPLIED VALUES:
EXTERNAL_PORT: 3489
andrew@andrew-xubuntulp:~$ helm get values promgra --all
COMPUTED VALUES:
EXTERNAL_IP: 192.168.11.133
EXTERNAL_PORT: 3489
GF_ADMIN_PASSWORD: HiGrafana
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
