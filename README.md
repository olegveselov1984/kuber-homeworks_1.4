# Домашнее задание к занятию «Сетевое взаимодействие в Kubernetes»

### Примерное время выполнения задания

120 минут

### Цель задания

Научиться настраивать доступ к приложениям в Kubernetes:
- Внутри кластера через **Service** (ClusterIP, NodePort).
- Снаружи кластера через **Ingress**.

Это задание поможет вам освоить базовые принципы сетевого взаимодействия в Kubernetes — ключевого навыка для работы с кластерами.
На практике Service и Ingress используются для доступа к приложениям, балансировки нагрузки и маршрутизации трафика. Понимание этих механизмов поможет вам упростить управление сервисами в рабочих окружениях и снизит риски ошибок при развёртывании.

------

## **Подготовка**
### **Чеклист готовности**
- Установлен Kubernetes (MicroK8S, Minikube или другой).
- Установлен `kubectl`.
- Редактор для YAML-файлов (VS Code, Vim и др.).

------

### Инструменты, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Инструкция](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download) по установке Minikube. 
3. [Инструкция](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)по установке kubectl.
4. [Инструкция](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools) по установке VS Code

### Дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

## **Задание 1: Настройка Service (ClusterIP и NodePort)**
### **Задача**
Развернуть приложение из двух контейнеров (`nginx` и `multitool`) и обеспечить доступ к ним:
- Внутри кластера через **ClusterIP**.
- Снаружи через **NodePort**.

### **Шаги выполнения**
1. **Создать Deployment** с двумя контейнерами:
   - `nginx` (порт `80`).
   - `multitool` (порт `8080`).
   - Количество реплик: `3`.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-deployment
  labels:
    app: main
spec:
  replicas: 3
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        env:
          - name: HTTP_PORT
            value: "80"        
        # ports:
        # - containerPort: 8080
      - name: network-multitool
        image: wbitt/network-multitool
        # ports:
        # - containerPort: 8085
        env:
          - name: HTTP_PORT
            value: "8080"
```


2. **Создать Service типа ClusterIP**, который:
   - Открывает `nginx` на порту `9001`.
   - Открывает `multitool` на порту `9002`.
  
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-mt-svc
spec:
  ports:
  - name: nginx-svc
    protocol: TCP
    port: 9001
    targetPort: 80
  - name: mt-svc
    protocol: TCP
    port: 9002
    targetPort: 8080
  selector:
    app: main 
  type: ClusterIP # Можно не писать, по умолчанию
```

3. **Проверить доступность** изнутри кластера:
```bash
 kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh
 curl <service-name>:9001 # Проверить nginx
 curl <service-name>:9002 # Проверить multitool
```

```
ubuntu@ubuntu:~/src/kuber/1.4/kuber-homeworks_1.4$ kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh
If you don't see a command prompt, try pressing enter.
/ #  curl nginx-mt-svc:9001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
/ #  curl nginx-mt-svc:9002
WBITT Network MultiTool (with NGINX) - netology-deployment-7ccf96f75-2g846 - 10.1.243.244 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
/ # 
```
<img width="1044" height="509" alt="image" src="https://github.com/user-attachments/assets/fc798756-d4a8-4c68-a3e2-75f3ea7dda32" />



4. **Создать Service типа NodePort** для доступа к `nginx` снаружи.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-mt-svc
spec:
  ports:
  - name: nginx-svc
    protocol: TCP
    port: 9001
    targetPort: 80
    nodePort: 30080
  - name: mt-svc
    protocol: TCP
    port: 9002
    targetPort: 8080
  selector:
    app: main 
  type: NodePort 
```

5. **Проверить доступ** с локального компьютера:
```bash
 curl <node-ip>:<node-port>
   ```
 или через браузер.
 
```
ubuntu@ubuntu:~/src/kuber/1.4/kuber-homeworks_1.4$ curl 127.0.0.1:30080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
 <img width="711" height="429" alt="image" src="https://github.com/user-attachments/assets/6ab43e89-4a7a-4399-bed3-454bc2e0e24c" />


### **Что сдать на проверку**
- Манифесты:
  - `deployment-multi-container.yaml`
  - `service-clusterip.yaml`
  - `service-nodeport.yaml`
- Скриншоты проверки доступа (`curl` или браузер).

---
## **Задание 2: Настройка Ingress**
### **Задача**
Развернуть два приложения (`frontend` и `backend`) и обеспечить доступ к ним через **Ingress** по разным путям.

### **Шаги выполнения**
1. **Развернуть два Deployment**:
   - `frontend` (образ `nginx`).
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-deployment-front
  labels:
    app: main-front
spec:
  replicas: 3
  selector:
    matchLabels:
      app: main-front
  template:
    metadata:
      labels:
        app: main-front
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        env:
          - name: HTTP_PORT
            value: "80"        
        # ports:
        # - containerPort: 8080
```

   - `backend` (образ `wbitt/network-multitool`).
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-deployment-back
  labels:
    app: main-back
spec:
  replicas: 3
  selector:
    matchLabels:
      app: main-back
  template:
    metadata:
      labels:
        app: main-back
    spec:
      containers:
      - name: network-multitool
        image: wbitt/network-multitool
        # ports:
        # - containerPort: 8085
        env:
          - name: HTTP_PORT
            value: "8080"
```


2. **Создать Service** для каждого приложения.
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-mt-svc-front
spec:
  ports:
  - name: nginx-svc
    protocol: TCP
    port: 9001
    targetPort: 80
    nodePort: 30080
  selector:
    app: main-front 
  type: NodePort 
```

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-mt-svc-back
spec:
  ports:
  - name: mt-svc
    protocol: TCP
    port: 9002
    targetPort: 8080
  selector:
    app: main-back 
  type: NodePort 
```


3. **Включить Ingress-контроллер**:
```bash
 microk8s enable ingress
   ```
4. **Создать Ingress**, который:
   - Открывает `frontend` по пути `/`.
   - Открывает `backend` по пути `/api`.
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-front-back
  annotations:  # ВАЖНО: Эта аннотация нужна для rewrite правил
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
#  - host: 127.0.0.1
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-mt-svc-front # УКАЖИТЕ: Имя frontend Service
            port:
              number: 9001
      - path: /api # КЛЮЧЕВОЙ ПУТЬ: API endpoint
        pathType: Prefix
        backend:
          service:
            name: nginx-mt-svc-back # УКАЖИТЕ: Имя backend Service
            port:
              number: 9002
```

5. **Проверить доступность**:
```bash
 curl <host>/
 curl <host>/api
   ```
 или через браузер.

```
ubuntu@ubuntu:~/src/kuber/1.4/kuber-homeworks_1.4$ curl 127.0.0.1/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
ubuntu@ubuntu:~/src/kuber/1.4/kuber-homeworks_1.4$ curl 127.0.0.1/api
WBITT Network MultiTool (with NGINX) - netology-deployment-back-c476f9bf5-4h6xg - 10.1.243.252 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```
 <img width="1050" height="478" alt="image" src="https://github.com/user-attachments/assets/554238d5-cfa7-4fe1-a999-1b191dd94a73" />


### **Что сдать на проверку**
- Манифесты:
  - `deployment-frontend.yaml`
  - `deployment-backend.yaml`
  - `service-frontend.yaml`
  - `service-backend.yaml`
  - `ingress.yaml`
- Скриншоты проверки доступа (`curl` или браузер).

---
## Шаблоны манифестов с учебными комментариями
### **1. Deployment (nginx + multitool)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: # ПРИМЕР: "multi-container-app"
spec:
  replicas: # ЗАДАНИЕ: Укажите количество реплик
  selector:
    matchLabels:
      app: # ДОПОЛНИТЕ: Метка для селектора
  template:
    metadata:
      labels:
        app: # ПОВТОРИТЕ метку из selector.matchLabels
    spec:
      containers:
 - name: # ЗАДАНИЕ: Название первого контейнера
        image: nginx
        ports:
 - containerPort: 80
 - name: multitool
        image: wbitt/network-multitool
        ports:
 - containerPort: 8080
        env:
 - name: HTTP_PORT
          value: "8080" # КЛЮЧЕВОЙ МОМЕНТ: Порт должен совпадать с containerPort
```
### **2. Ingress (для frontend и backend)**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: # ЗАДАНИЕ: Придумайте имя, допустим example-ingress
  annotations:  # ВАЖНО: Эта аннотация нужна для rewrite правил
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
 - http:
      paths:
 - path: /
        pathType: Prefix
        backend:
          service:
            name: # УКАЖИТЕ: Имя frontend Service
            port:
              number: 80
 - path: /api # КЛЮЧЕВОЙ ПУТЬ: API endpoint
        pathType: Prefix
        backend:
          service:
            name: # УКАЖИТЕ: Имя backend Service
            port:
              number: 80
```
---

## **Правила приёма работы**
1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

## **Критерии оценивания задания**
1. Зачёт: Все задачи выполнены, манифесты корректны, есть доказательства работы (скриншоты).
2. Доработка (на доработку задание направляется 1 раз): основные задачи выполнены, при этом есть ошибки в манифестах или отсутствуют проверочные скриншоты.
3. Незачёт: работа выполнена не в полном объёме, есть ошибки в манифестах, отсутствуют проверочные скриншоты. Все попытки доработки израсходованы (на доработку работа направляется 1 раз). Этот вид оценки используется крайне редко.

## **Срок выполнения задания**  
1. 5 дней на выполнение задания.
2. 5 дней на доработку задания (в случае направления задания на доработку).
