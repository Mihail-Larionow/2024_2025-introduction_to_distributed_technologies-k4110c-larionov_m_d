University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2024/2025  
Group: K4110c  
Author: Larionov Mikhail Dmitrievich  
Lab: Lab1  
Date of create: 16.11.2024  
Date of finished: 16.11.2024  

# Лабораторная работа №1 "Установка Docker и Minikube, мой первый манифест."

### Цель работы

Ознакомиться с инструментами _Minikube_ и _Docker_, развернуть свой первый "под".

## Ход работы

Для успешного выполнения необходимо, чтобы в системе были установлены _docker_, _kubectl_ и _minikube_.

Запускаем minikube при помощи команды:

```
minikube start --driver=docker
```

В нем мы попробуем запустить менеджер для хранения и распространения секретов **Vault**.  
Для этого создадим файл vault.yml:

```
apiVersion: v1
kind: Pod
metadata:
  name: vault
  labels:
    name: vault
spec:
  containers:
  - name: vault
    image: vault:1.13.3
    ports:
      - containerPort: 8200
```

Создадим под при помощи команды:

```
kubectl apply -f vault.yml
```

<p align="center"><img src="https://github.com/user-attachments/assets/6b9aebb5-b961-4014-84db-2243bd99cb8d"/></p>

Запустим сервис:

```
minikube kubectl -- expose pod vault --type=NodePort --port=8200
```

Прокинем порт компьютера в контейнер, чтобы попасть в **Vault** по [ссылке](http://localhost:8200):

```
minikube kubectl -- port-forward service/vault 8200:8200
```

<p align="center"><img src="https://github.com/user-attachments/assets/c26229c3-e34c-4d6b-b700-ff254e2cd6fd"/></p>

Посмотрим логи и найдем в них токен, которым воспользуемся при входе в систему:

```
kubectl logs vault
```

<p align="center"><img src="https://github.com/user-attachments/assets/3b031a12-09fb-4a2d-b00c-3244499d281c"/></p>

## Вывод

В ходе лабораторной работы мы познакомились с инструментами _Minikube_ и _Docker_, а также развернули свой _**Vault**_ под.

## Ответы на вопросы

1. Что сейчас произошло и что сделали команды указанные ранее?  
   В kubernetes-кластере minikube был создан под с контейнером, в котором было запущено хранилище секретов Vault. Для связи с сервисом мы прокинули порт 8200
   у локального хоста через такой же порт у контейнера.

3. Где взять токен для входа в Vault?  
   Токен для входа в Vault можно найти в логах пода.

## Схема организации контейнеров и сервисов

<p align="center"><img src="https://github.com/user-attachments/assets/2e834a20-94f3-4fa0-ba08-8ac766d942b0"/></p>

