University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2024/2025
Group: K4111c
Author: Burak Petr Vasilievich
Lab: Lab1
Date of create: 20.09.2024
Date of finished: 31.09.2024

# Лабораторная работа №1 "Установка Docker и Minikube, мой первый манифест."

## Ход решения:
1. Предварительно был установлен и запущен Docker а также установлен и запущен Minikube согласно инструкции.
- `minikube start` произошел запуск minikube, который представляет из себя кластер из одной ноды.
### Запуск пода
2. После запуска кластера и проверки подключения, был создан манифест vault.yaml. Для развертывания пода была использована следущая команда:
- `kubectl apply -f vault.yaml` - данная команда сохраняет написанный ранее манифест и деплоит под.
4. Убедились, что под был успешно запущен при помощи команды `kubectl get pods`
5. Манифест:

```apiVersion: v1
kind: Pod
metadata:
  name: vault
  labels:
    app: vault
spec:
  containers:
    - name: vault
      image: hashicorp/vault:latest
      ports:
        - containerPort: 8200
```

6. Создаём сервис доступа к контейнеру с помощью команды minikube kubectl -- expose pod vault --type=NodePort --port=8200 - данная команда создает сервис для открытия доступа к приложению по порту 8200.