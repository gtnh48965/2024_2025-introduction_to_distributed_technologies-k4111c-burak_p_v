University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)

Year: 2023/2024

Group: K4111c

Author: Burak Petr Vasilievich

Lab: Lab3

Date of create: 09.12.2024

Date of finished: 13.12.2024

# Ход работы
## 1. Подготовка манифестов
Сначала необходимо было создать манифест configMap, содержащий следующий набор переменных:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-configmap
data:
  react_app_user_name: "Burak Petr"
  react_app_company_name: "ITMO ICT"
```

Во-вторых, необходимо было создать манифест ReplicaSet.

* контейнер - **ifilyaninitmo/itdt-contained-frontend:master**
* порт - **3000**
* количество реплик - **2**
* следует использовать переменные среды из ConfigMap
    
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend-replicaset
  labels:
    app: lab3-frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: lab3-frontend
  template:
    metadata:
      labels:
        app: lab3-frontend
    spec:
      containers:
      - name: frontend-container
        image: ifilyaninitmo/itdt-contained-frontend:master
        ports:
        - containerPort: 3000
        env:
        - name: REACT_APP_USERNAME
          valueFrom:
            configMapKeyRef:
              name: frontend-configmap
              key: react_app_user_name
        - name: REACT_APP_COMPANY_NAME
          valueFrom:
            configMapKeyRef:
              name: frontend-configmap
              key: react_app_company_name
```

В-третьих, необходимо было создать сервис для раскрытия pod-ов:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  labels:
    app: lab3-frontend
spec:
  type: NodePort
  selector:
    app: lab3-frontend
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 30333
```

В-четвертых, необходимо было создать вход:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-ingress
spec:
  tls:
  - hosts:
      - burakfrontend
    secretName: lab3-tls
  rules:
  - host: burakfrontend
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 3000
```

## 2. Создание сертификата TLS
Для создания TLS-сертификата были использованы следующие команды:
```
openssl genrsa -out ca.key -2048
openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout ca.key -out ca.crt
```

## 3. Настройка minikube и среды
Из-за использования Docker в качестве драйвера minikube потребовалось добавить несколько дополнений для minikube:
```
minikube addons enable ingress
minikube addons enable ingress-dns
```

![img.png]()

Также следует отредактировать файл hosts , как показано ниже:
```
127.0.0.1 burakfrontend
```

При создании Ingress необходимо будет выполнить следующую команду:
```
minikube tunnel
```

## 4. Применение манифестов
Применение манифеста (+ создание секрета TLS):

![img_1.png]()

## 5. Доступ к приложению
Для доступа к приложению необходимо было перейти по следующей ссылке:
```
https://burakfrontend
```

![img_4.png]()

Информация о сертификате:

![img_3.png](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR3/image/result.png)

## 6. Общая архитектура
![img.png](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR3/image/totalСertificate.png)
