### Задание 1: подготовить тестовый конфиг для запуска приложения
- под содержит в себе 2 контейнера — фронтенд, бекенд
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№60$ kubectl get -n stage pods frontwithback-655fcf479-xjsxr -o jsonpath='{range .spec.containers[*]}{.name}{"\n"}{end}'
backend
frontend
```

- регулируется с помощью deployment фронтенд и бекенд
- - Манифест [deployment](./stage/Deployment.yml)

- база данных — через statefulset
- - Манифест [StorageClass](./stage/StorageClass.yml)
- - Манифест [PersistentVolumeClaim.](./stage/PersistentVolumeClaim.yml)
- - Манифест [PersistentVolume](./stage/PersistentVolume.yml)
- - Манифест [StatefulSet](./stage/StatefulSet.yml)

### Задание 2: подготовить конфиг для production окружения
- каждый компонент (база, бекенд, фронтенд) запускаются в своем поде, регулируются отдельными deployment’ами
- - Манифест [DeploymentBackend](./prod/DeploymentBackend.yml)
- - Манифест [DeploymentFrontend](./prod/DeploymentFrontend.yml)
- - Манифест [StatefulSet](./prod/StatefulSet.yml)
- для связи используются service (у каждого компонента свой)
- - Манифесты сервисов описаны в Манифестах предыдущего пункта
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№60$ kubectl get -n prod service -o wide
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE   SELECTOR
backend         ClusterIP   10.233.26.237   <none>        9000/TCP   42m   app=backend
front           ClusterIP   10.233.60.117   <none>        8000/TCP   75m   app=front
postgres-prod   ClusterIP   10.233.9.145    <none>        5432/TCP   26m   app=postgres-prod
```
- - в окружении фронта прописан адрес сервиса бекенда;
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№60$ kubectl exec -it -n prod front-578cd54fcd-tftms -- env | grep BACKEND
BACKEND_SERVICE_PORT=9000
BACKEND_PORT_9000_TCP=tcp://10.233.30.94:9000
BACKEND_PORT_9000_TCP_ADDR=10.233.30.94
BACKEND_SERVICE_PORT_BACKEND=9000
BACKEND_PORT=tcp://10.233.30.94:9000
BACKEND_PORT_9000_TCP_PROTO=tcp
BACKEND_SERVICE_HOST=10.233.30.94
BACKEND_PORT_9000_TCP_PORT=9000
```
- - в окружении бекенда прописан адрес сервиса базы данных.
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№60$ kubectl exec -it -n prod backend-55d99c978c-tk2ld -- env | grep POSTGRES
POSTGRES_PROD_PORT_5432_TCP_PORT=5432
POSTGRES_PROD_SERVICE_PORT=5432
POSTGRES_PROD_SERVICE_PORT_DB=5432
POSTGRES_SERVICE_PORT=5432
POSTGRES_SERVICE_PORT_DB=5432
POSTGRES_PROD_SERVICE_HOST=10.233.9.145
POSTGRES_PROD_PORT_5432_TCP_PROTO=tcp
POSTGRES_PORT_5432_TCP_PORT=5432
POSTGRES_PORT_5432_TCP_PROTO=tcp
POSTGRES_PROD_PORT=tcp://10.233.9.145:5432
POSTGRES_PROD_PORT_5432_TCP=tcp://10.233.9.145:5432
POSTGRES_PROD_PORT_5432_TCP_ADDR=10.233.9.145
vlad@home:~/Documents/Netology/DevOps/dz_№60$ 
```

### Для проверки использовался контейнер multitool

Манифест [multitool](./prod/DaemonSet.yml)
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№60$ kubectl exec -it -n prod multitool-646475c984-7npjl -- /bin/bash
```
#### Проверка
- Frontend
```commandline
bash-5.1# curl front:8000
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Список</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="/build/main.css" rel="stylesheet">
</head>
<body>
    <main class="b-page">
        <h1 class="b-page__title">Список</h1>
        <div class="b-page__content b-items js-list"></div>
    </main>
    <script src="/build/main.js"></script>
</body>
</html>bash-5.1# 
```
- Backend
```commandline
</html>bash-5.1# curl backend:9000
{"detail":"Not Found"}
bash-5.1# 

```

- База данных
```commandline
bash-5.1# psql -U postgres -h postgres-prod 
Password for user postgres: 
psql (13.4, server 13.8)
Type "help" for help.

postgres=# 
```