University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2024/2025  
Group: K4110c  
Author: Larionov Mikhail Dmitrievich  
Lab: Lab2  
Date of create: 16.11.2024  
Date of finished: 16.11.2024  

# Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса."

### Цель работы

Ознакомиться с типами "контроллеров" развертывания контейнеров, сетевыми сервисами и развернуть свое веб приложение.

## Ход работы

Создадим конфигурацию _deployment_ с 2 репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend), в которые передадим переменные `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME`:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: itdt-contained-frontend
  labels:
    name: itdt-contained-frontend
spec:
  selector:
    matchLabels:
      app: itdt-contained-frontend
  replicas: 2
  template:
    metadata:
      labels:
        app: itdt-contained-frontend
    spec:
      containers:
        - name: itdt-contained-frontend
          image: ifilyaninitmo/itdt-contained-frontend:master
          env:
            - name: REACT_APP_USERNAME
              value: Mikhail Larionov
            - name: REACT_APP_COMPANY_NAME
              value: ITMO
```

Применим ее:

```
minikube kubectl -- apply -f itdt-contained-frontend.yml
```

<p align="center"><img src="https://github.com/user-attachments/assets/4854683f-56da-449e-aa6e-2ea8523ae5af"/></p>

Для доступа к получившимся подам напишем конфигурацию сервиса:

```
apiVersion: v1
kind: Service
metadata:
  name: itdt-contained-frontend-service
spec:
  type: LoadBalancer
  ports:
    - targetPort: 3000
      port: 3000
      nodePort: 30300
  selector:
    app: itdt-contained-frontend
```

Как можно заметить, в качестве сервиса, был использован **LoadBalancer**, который распределяет нагрузку сети между репликами, а также обеспечивает высокую доступность и маштабируемость.

Можно было, конечно, использовать **NodePort**, но данный сервис используется скорее для тестирования или разработки. Он, так сказать, внутренний балансировщик нагрузки, когда **LoadBalancer** — внешний.

Применим конигурацию:

```
minikube kubectl -- create -f itdt-contained-frontend-service.yml
```

В браузере откроем две вкладки, в каждом из которых введем _ip_-адрес нашего кластера, а также внешний порт сервиса.

Первая вкладка:

<p align="center"><img src="https://github.com/user-attachments/assets/e1dd530c-3d08-4419-8930-4971263eb310"/></p>

Вторая вкладка:

<p align="center"><img src="https://github.com/user-attachments/assets/25dbae24-a8f1-41ca-9354-c14cb547b945"/></p>

Мы видим, что переменные `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME` в контейнерах совпадают, однако их имена и _ip_-адреса различны. Так происходит, потому что **LoadBalancer** старается равномерно распределять нагрузку на поды.

Также посмотрим логи каждого из них.

Первый под:

<p align="center"><img src="https://github.com/user-attachments/assets/34121e5c-2adb-4b6e-88fc-fbf461413f07"/></p>

Второй под:

<p align="center"><img src="https://github.com/user-attachments/assets/70f36d93-9584-46c7-a3c1-4e899d680189"/></p>

## Вывод

В ходе лабораторной работы мы познакомились с типами "контроллеров" развертывания контейнеров, сетевыми сервисами и развернули свое веб приложение, тем самым закрепив знания на практике.

## Схема организации контейнеров и сервисов

<p align="center"><img src="https://github.com/user-attachments/assets/d59e8972-e151-4397-8b10-536bab95fb6d"/></p>

