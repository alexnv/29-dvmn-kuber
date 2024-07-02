# Django site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри конейнера Django запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).


## Серверная инфраструктура: [edu-sleepy-engelbart](https://sirius-env-registry.website.yandexcloud.net/edu-sleepy-engelbart.html)
Подключиться к кластеру можно например через [Lens](https://k8slens.dev/)

Установите [Helm](https://helm.sh/)

Добавьте Bitnami
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
## Деплой приложения

Примените манифесты:
```
kubectl apply -f simple-pod.yaml
```


## [Ссылка на сервер](https://edu-sleepy-engelbart.sirius-k8s.dvmn.org/)