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
### Установка secrets - переменных окружения и SSL сертификата

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

#### Переменные окружения
В манифест `secrets.yaml`, надо прописать следующие данные:
 - необходимо генерировать случайную строку для переменной `SECRET_KEY` и задать `DEBUG` режим.
 - указать в списке `ALLOWED_HOSTS` ваш домен.

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
