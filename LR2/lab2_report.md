University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2024/2025
Group: K4111c
Author: Burak Petr Vasilievich
Lab: Lab2
Date of create: 20.10.2024
Date of finished: 31.10.2024

# Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса.""

# Progress of work
## 1. Подготовка манифеста развертывания
Сначала необходимо было создать манифест развертывания.
Требования представлены ниже:
* контейнер - **ifilyaninitmo/itdt-contained-frontend:master**
* контейнерный порт - **3000**
* количество реплик - **2**
* должны быть установлены переменные среды:
    * REACT_APP_USERNAME
    * REACT_APP_COMPANY_NAME

Используя приведенные выше требования, был создан манифест развертывания:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend_pod
  template:
    metadata:
      labels:
        app: frontend_pod
    spec:
      containers:
      - name: frontend-container
        image: ifilyaninitmo/itdt-contained-frontend:master
        ports:
        - containerPort: 3000
        env:
        - name: REACT_APP_USERNAME
          value: Peter
        - name: REACT_APP_COMPANY_NAME
          value: ITMO_Itmo
```

## 2. Применение манифеста
Для применения манифеста необходимо использовать следующую команду:
```
kubectl apply -f <file>
```
Итак, манифест был применен:

![image](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR2/image/img_1.png)
![image](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR2/image/img_2.png)
Кроме того, была распечатана некоторая информация, например информация о развертывании и модулях.

## 3. Раскрытие развертывания
После применения было открыто развертывание, и для доступа к контейнеру с хост-машины использовалась переадресация портов:

![image](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR2/image/img_3.png)

Было решено использовать тип сервиса **LoadBalancer**. В результате открылась следующая страница:

![image](https://github.com/gtnh48965/2024_2025-introduction_to_distributed_technologies-k4111c-burak_p_v/blob/main/LR2/image/img_4.png)

После обновления ничего не изменилось. Кажется, не хватает загрузки пода, 
поэтому балансировщик нагрузки решает не пересылать запросы на другой модуль.

Логи подов тоже такие же:
```
Builing frontend
build finished
Server started on port 3000
```

## 4. Общая архитектура
На рисунке ниже описаны сущности, которые используются в текущей лабораторной работе.
<!-- ![img_3.png](resources/img_3.png) -->
