University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2023/2024

Group: K4111c

Author: Burak Petr Vasilievich

Lab: Lab4

Date of create: 10.12.2024

Date of finished: 20.12.2024

# Ход работы
## 1. Ход работы
Ниже представлен пошаговый ход работы

### 1.1 Настройка среды
`minikube` в данной работе запускается с помощью следующей команды:
```bash
minikube start --network-plugin=cni --cni=calico --nodes 2 -p multinode-demo
```

После успешного запуска проверим, что создалось действительно две ноды с помощью команды
```bash
kubectl get nodes
```
![1.png](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR4/image/getNodes.png)

Также не лишним будет проверить количество подов `calico`.
Их число должно совпадать с количеством нод.
```bash
kubectl get pods -l k8s-app=calico-node -A
```
![2.png](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR4/image/k8s-app=calico-node.png)

### 1.3 Пометка нод

Каждую ноду в соответствии с заданием пометим по признаку стойки. Так, кажется, будет логичнее.

В соответствии с заданием помечается каждая нода по географическому расположению

Для назначения IP адресов в Calico необходимо написать манифест для IPPool ресурса.

С помощью IPPool можно создать IP-pool (блок IP-адресов), который выделяет IP-адреса только для узлов с определенной меткой (label).

С помощью следующей команды, назначаются метки узлам :

Пометка делается с помощью команды
```bash
kubectl label nodes multinode-demo ra=a01
kubectl label nodes multinode-demo-m02 ra=a02
```
[3.png](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR4/image/kubectlLabel.png)

## 2. Настройка calico
Далее из официальной документации Calico берется шаблон манифеста IPPool.

Проверим у calico дефолтный ippool:
```bash
calicoctl get ippool -o wide --allow-version-mismatch
```
![4.png](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR4/image/mismatch.png)

И удаляем его, заменяя на нашу конфигурацию:
```yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: ra-a01-pool
spec:
  cidr: 192.168.0.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: ra == "a01"

---

apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: ra-a02-pool
spec:
  cidr: 192.168.1.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: ra == "a02"
```

![5.png](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR4/image/deleteIppools.png)

![6.png](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR4/image/applyIppool.png)

Проверяем созданные пулы:

![7.png](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR4/image/getIppool.png)

## 3. Deployment и Service

И теперь мы создаем деплоймент с 2 репликами, NodePort и подключаем configMap как в предыдущих ЛР.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lab4-deployment
  labels:
    app: lab4-frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend-l4
  template:
    metadata:
      labels:
        app: frontend-l4
    spec:
      containers:
        - name: frontend-l4
          image: ifilyaninitmo/itdt-contained-frontend:master
          env:
            - name: REACT_APP_USERNAME
              valueFrom:
                configMapKeyRef:
                  name: configmap-demo
                  key: user

            - name: REACT_APP_COMPANY_NAME
              valueFrom:
                configMapKeyRef:
                  name: configmap-demo
                  key: company

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-demo
data:
  user: "Nikita"
  company: "ITMO University"
```


![8.png](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR4/image/applyManifest.png)
Видим что сервис, конфиг мапа и деплоймент созданы.

```bash
kubectl expose deployments lab4-deployment --type=NodePort --port=3000
```

![9.png](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR4/image/kubectlExpose.png)
И теперь пробросим порты
```bash
kubectl port-forward lab4-deployment 4000:4000
```
и переходим по локальному хосту на порт:

## 4. Ping подов
В конце проверили, что поды пингуют друг друга:
Результат:
![10.png](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR4/image/ping.png)

### 5. Диаграмма развертывания
![]()



