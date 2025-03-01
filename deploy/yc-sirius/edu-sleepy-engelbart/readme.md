# # Запуск в кластере Yandex Cloud

(параметры сервера)[https://sirius-env-registry.website.yandexcloud.net/edu-sleepy-engelbart.html]

### Сборка приложения

Для сборки и отправки в репозитарий [hub.docker.com](hub.docker.com) необходимо запустить, в папке `backend_main_django` следующий скрипт (заменив `namespace_in_dockerhub` на свой логин в `dockerhub`).

Создание образа:
```shell
docker build -t <namespace_in_dockerhub>/k8s-dvmn:lastest .
```
Загрузка в dockerhub:
```shell
docker pull <namespace_in_dockerhub>/k8s-dvmn:lastest
```

## Серверная инфраструктура: [edu-sleepy-engelbart](https://sirius-env-registry.website.yandexcloud.net/edu-sleepy-engelbart.html)

Подключиться к кластеру можно например через [Lens](https://k8slens.dev/)

Установите [Helm](https://helm.sh/)

Добавьте Bitnami
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Разверните PostgreSQL в кластере, при необходимости:
```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install postgres bitnami/postgresql
```
В качестве хоста БД используйте:
```
postgres.default.svc.cluster.local
```

Добавьте его в `host` файла `secrets-pg.yaml`.

Сохраните пароль в переменной окружения `POSTGRES_PASSWORD`:

```bash
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgress-secret -o jsonpath="{.data.password}" | base64 -d)
```

Чтобы подключиться к вашей базе данных, выполните следующую команду:

```bash
kubectl run postgres-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:16.1.0-debian-11-r20 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host postgres-postgresql -U postgres -d postgres -p 5432
```

Создайте БД и пользователя в PostgreSQL:

```sql
create database Имя_вашей_бд;
create user ваш_логин with encrypted password 'пароль_пользователя';
grant all privileges on database Имя_вашей_бд to ваш_логин;
ALTER DATABASE Имя_вашей_бд OWNER TO ваш_логин;
```

Замените `Имя_вашей_бд` и `ваш_логин` на соответствующие значения.

Создайте манифест файл secrets-pg.yaml:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgress-secret
  namespace: edu-sleepy-engelbart
  labels:
    app.kubernetes.io/name: postgress-secret
    app.kubernetes.io/env: dev
type: Opaque
stringData:
  username: ""
  password: ""
  host: ""
  port: ""
  name: ""
```


Создайте манифест файл secrets.yaml:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: django-secret
  namespace: edu-sleepy-engelbart
  labels:
    app.kubernetes.io/name: django-secret
    app.kubernetes.io/env: dev
type: Opaque
stringData:
  SECRET_KEY: your_allowed-hosts
  ALLOWED_HOSTS: secret-key
  DEBUG: "False"
  DATABASE_URL: "postgres://$(username):$(password)@$(host):$(port)/$(name)?sslmode=verify-full&sslrootcert=/opt/.postgresql/root.crt"
```

#### Установка secrets - переменных окружения и SSL сертификата

Проверьте, возможно уже есть `Secret` с сертификатом для подключения к базе данных, в этом случае можно пропустить шаг с установкой сертификата и созданием Secret:

```shell
kubectl get secret
kubectl get secret pg-root-cert -o yaml
```

#### Установка рутового SSL сертификата для подключения к Postgres
Для верификации сервера при шифровании соединения необходимо поместить в secrets рутовый сертификат Яндекса. 
Для этого его необоходимо сначала [скачать](https://storage.yandexcloud.net/cloud-certs/CA.pem) со страницы [справки](https://yandex.cloud/ru/docs/managed-postgresql/operations/connect). 
Затем поместить в secrets:
```shell
kubectl create secret generic pg-root-cert --from-file=<path_to_cert>root.crt
```

#### При первом запуске проекта создайте суперпользователя:
Посмотрите имя подов приложения:
```shell-session
kubectl get pods
```
Подключитесь к любому из подов:
```shell-session
kubectl exec -it <имя-пода> -- /bin/bash
```

Создайте суперпользователя:
```shell-session
python manage.py createsuperuser
```

### Запуск приложения

Перед запуском приложения нужно применить миграции. Для применения миграций к базе при обновлении приложения используется следующая команда:
```shell
kubectl apply -f ./secrets.yaml
kubectl apply -f ./migrate.yml
```

Для запуска приложения нужно создать объект Deployment:
```shell
kubectl apply -f ./deployment.yml
kubectl apply -f ./service.yaml
kubectl apply -f ./clearsessions.yaml
```

Сайт будет доступен по адресу: [https://edu-sleepy-engelbart.sirius-k8s.dvmn.org](https://edu-sleepy-engelbart.sirius-k8s.dvmn.org/)
