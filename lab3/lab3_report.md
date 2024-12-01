University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2024/2025  
Group: K4110c  
Author: Larionov Mikhail Dmitrievich  
Lab: Lab3  
Date of create: 30.11.2024  
Date of finished: 30.11.2024  

# Лабораторная работа №3 "Сертификаты и "секреты" в Minikube, безопасное хранение данных."

### Цель работы

Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube. 

## Ход работы

Первым делом создадим _ConfigMap_, в котором зададим значения переменных `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME`:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: itdt-contained-frontend-configmap
  labels:
    name: itdt-contained-frontend-configmap
data:
  REACT_APP_USERNAME: Mikhail Larionov
  REACT_APP_COMPANY_NAME: ITMO
```

Применим:

```
minikube kubectl create -f configmap.yml
```

<p align="center"><img src="https://github.com/user-attachments/assets/5ce38812-8e1a-4ba2-8783-617d8e462efa"/></p>

Теперь создадим конфигурацию _ReplicaSet_, в которой возьмем параметры для подов из ранее созданного _ConfigMap_:

```
apiVersion: apps/v1
kind: ReplicaSet
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
              valueFrom:
                configMapKeyRef:
                  name: itdt-contained-frontend-configmap
                  key: REACT_APP_USERNAME
            - name: REACT_APP_COMPANY_NAME
              valueFrom:
                configMapKeyRef:
                  name: itdt-contained-frontend-configmap
                  key: REACT_APP_COMPANY_NAME
```

<p align="center"><img src="https://github.com/user-attachments/assets/bd984295-f61d-442f-880c-98181e4f193f"/></p>

Можно заметить, что в прошлой работе в похожей ситуации мы использовали _Deployment_. На самом деле, под капотом _Deployment_ также использует _ReplicaSet_, но помимо репликации подов он позволяет контролировать версии продукта.

Для дальнейшей работы с _Ingress_ нам необходимо включить соответствующий аддон.

```
minikube addons enable ingress
```

Перед тем как подписывать TLS сертификат, сгенерируем приватный ключ, а также запрос на подпись сертификата:

```
openssl genrsa -out mikhail.key 4096
openssl req -key mikhail.key -new -out mikhail.csr
```

После чего подпишем наш сертификат:

```
openssl x509 -signkey mikhail.key -in mikhail.csr -req -days 365 -out mikhail.crt
```

И создадим для него _Secret_ в _minikube_:

```
kubectl create secret tls mikhail-tls --cert=mikhail.crt --key=mikhail.key
```

Теперь его можно использовать при конфигурировании _Ingress_:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: itdt-contained-frontend-ingress
  labels:
    name: itdt-contained-frontend-ingress
spec:
  rules:
    - host: mikhail-larionov.ru
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: itdt-contained-frontend-service
                port:
                  number: 3000
  tls:
    - hosts:
        - mikhail-larionov.ru
      secretName: mikhail-tls
```

Можно заметить, что в качестве [сервиса](https://github.com/Mihail-Larionow/2024_2025-introduction_to_distributed_technologies-k4110c-larionov_m_d/blob/main/lab3/service.yml) был использован тот же _LoadBalacncer_, что и в прошлой лабораторной работе.

Применим данную конфигурацию.

<p align="center"><img src="https://github.com/user-attachments/assets/0cfe1894-7960-4c10-83fd-a609d4135655"/></p>

Чтобы иметь возможность заходить по FQDN, добавим запись о нем в `/etc/hosts` _(ip-адрес можно легко узнать, воспользовавшись командой `echo $(minikube ip)`)_.  

Наконец, введем в браузер _https://mikhail-larionov.ru_ и увидим уже знакомую нам страницу:

<p align="center"><img src="https://github.com/user-attachments/assets/1064d905-5704-4a74-a20b-ce0e1296c93c"/></p>

Также, мы можем посмотреть информацию о сертификате ресурса, которая удивительным образом совпадает с той, что мы указывали при его создании.

<p align="center"><img src="https://github.com/user-attachments/assets/60946785-0413-415c-bce7-7166f360a013"/></p>

## Вывод

В ходе лабораторной работы мы познакомились с сертификатами и "скеретами" в Minikube, а также с правилами безопасного храгнения данных, закрепив знания на практике.

## Схема организации контейнеров и сервисов

<p align="center"><img src="https://github.com/user-attachments/assets/74fc794d-5fd4-48ee-8943-95198244249a"/></p>

