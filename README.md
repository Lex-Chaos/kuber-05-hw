# Домашняя работа к занятию «Сетевое взаимодействие в K8S. Часть 2» - Боровик А.А.
### Цель задания
В тестовой среде Kubernetes необходимо обеспечить доступ к двум приложениям снаружи кластера по разным путям.

------
### Чеклист готовности к домашнему заданию
1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------
### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания
1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Service.
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.
  
------
### Задание 1. Создать Deployment приложений backend и frontend
1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт.
2. Создать Deployment приложения _backend_ из образа multitool.
3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера.
4. Продемонстрировать, что приложения видят друг друга с помощью Service.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

## Ответ:

[Манифест frontend-deployment](https://github.com/Lex-Chaos/kuber-05-hw/blob/main/files/frontend-deployment.yml):

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

[Манифест backend-deployment](https://github.com/Lex-Chaos/kuber-05-hw/blob/main/files/backend-deployment.yml):

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: multitool
  template:
    metadata:
      labels:
        app: multitool
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 80
```

[Манифест frontend-service](https://github.com/Lex-Chaos/kuber-05-hw/blob/main/files/frontend-service.yml):

```yml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: default
spec:
  selector:
    app: nginx
  type: ClusterIP
  ports:
  - name: nginx
    protocol: TCP
    port: 9001
    targetPort: 80
```

[Манифест backend-service](https://github.com/Lex-Chaos/kuber-05-hw/blob/main/files/backend-service.yml):

```yml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: default
spec:
  selector:
    app: multitool
  type: ClusterIP
  ports:
  - name: multitool
    protocol: TCP
    port: 9002
    targetPort: 80
```
  
Вывод доступа к приложениям:

![curl](https://github.com/Lex-Chaos/kuber-05-hw/blob/main/img/Test1.png):

  ------
### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера
1. Включить Ingress-controller в MicroK8S.
2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_.
3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
4. Предоставить манифесты и скриншоты или вывод команды п.2.

## Ответ:

[Манифест ingress](https://github.com/Lex-Chaos/kuber-05-hw/blob/main/files/ingress.yml):

```yml
apiVersion: networking.k8s.io/v1 
kind: Ingress
metadata:
  name: ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: / 
spec:
  ingressClassName: "nginx"
  rules:
  - host: front-end.my
    http: 
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port: 
              number: 80
      - path: /api
        pathType: Exact
        backend:
          service:
            name: backend-service 
            port: 
              number: 80
```

Включил ingress:

![ingress](https://github.com/Lex-Chaos/kuber-05-hw/blob/main/img/Test2-1.png)

Вывод доступа через браузер:

![browser](https://github.com/Lex-Chaos/kuber-05-hw/blob/main/img/Test2-2.png)

------
### Правила приема работы
1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
  
------
