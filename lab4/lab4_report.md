University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2024/2025  
Group: K4110c  
Author: Larionov Mikhail Dmitrievich  
Lab: Lab4  
Date of create: 03.12.2024  
Date of finished: 03.12.2024  

# Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS."

### Цель работы

Познакомиться с CNI Calico и функцией IPAM Plugin, изучить особенности работы CNI и CoreDNS.

## Ход работы

Запустим _minikube_ с плагином `CNI=calico` в режиме `Multi-Node Clusters` с двумя нодами:

```
minikube start --network-plugin=cni --cni=calico --nodes=2 --driver=docker
```

Проверим, что ноды и поды успешно развернулись:

<p align="center"><img src="https://github.com/user-attachments/assets/efb9e058-62ce-4582-8a7b-19eb7eedead6"/></p>

Удалим стандартный _IPPool_ и добавим два новых: 

```
apiVersion: projectcalico.org/v3
kind: IPPool
metadata: 
  name: russia-ipv4-ippool
spec:
  nodeSelector: zone == "russia"
  cidr: 192.168.0.0/24
  ipipMode: Always
  natOutgoing: true
```

_Пример конфигурации IPPool для зоны russia._

```
kubectl delete ippool default-ipv4-ippool
calicoctl create -f russia-ippool.yml --allow-version-mismatch
calicoctl create -f europe-ippool.yml --allow-version-mismatch
```

<p align="center"><img src="https://github.com/user-attachments/assets/8313132a-2b0c-467d-ab3e-36ade714fe11"/></p>

Укажем для каждой ноды `label`:

```
kubectl label node minikube zone=russia
kubectl label node minikube-m02 zone=europe
```

Из прошлых работ возьмем [_Deployment_](https://github.com/Mihail-Larionow/2024_2025-introduction_to_distributed_technologies-k4110c-larionov_m_d/blob/main/lab4/itdt-contained-frontend-deployment.yml) с 2 репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend), в которые передадим переменные `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME`.

Также поступим и с сервисом, взяв за основу [_LoadBalancer_](https://github.com/Mihail-Larionow/2024_2025-introduction_to_distributed_technologies-k4110c-larionov_m_d/blob/main/lab4/itdt-contained-frontend-service.yml).  

<p align="center"><img src="https://github.com/user-attachments/assets/61858f30-433b-4775-96cb-e5d23d3ebd31"/></p>

Через браузер подключимся к нашим контейнерам.

_Контейнер 1:_ 

<p align="center"><img src="https://github.com/user-attachments/assets/727b0333-a621-48c2-bb66-b24b95669ce9"/></p>

_Контейнер 2:_ 

<p align="center"><img src="https://github.com/user-attachments/assets/d16bc0e5-795c-4d6e-aa7b-9bfdb0e561e7"/></p>

Как видим, название и адрес контейнеров различается, причем `ip` соответствует пулу адресов ноды. Убедимся в этом еще раз: 

<p align="center"><img src="https://github.com/user-attachments/assets/db544286-9204-4f39-9e72-98e1291a1c52"/></p>

Проверим доступность подов, для этого подключимся к каждому из них и попробуем достучаться до соседа:

<p align="center"><img src="https://github.com/user-attachments/assets/9701b1a2-8752-4da0-ae3a-d808d75215ba"/></p>

<p align="center"><img src="https://github.com/user-attachments/assets/f6f7c2e3-68eb-4ede-8f87-3fbcd8485054"/></p>

Можно заметить, что пинг успешно проходит, что не может не радовать.

## Вывод

В ходе лабораторной работы мы познакомились с CNI Calico и функцией IPAM Plugin, а также изучили особенности работы CNI и CoreDNS и закрепили знания на практике.

## Схема организации контейнеров и сервисов

<p align="center"><img src="https://github.com/user-attachments/assets/a2a73cdc-06b4-4bbe-b9f0-8bc8e3439899"/></p>

