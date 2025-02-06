## Что потребуется

Потребуется image Django. Репозиторий доступен по [ссылке](https://github.com/devmanorg/k8s-test-django).
Можно собрать прямо в minikube, либо просто добавить image в кэш: 
```
minikube cache add django_app:latest
```

Для запуска minikube + virtualbox на Windows необходимо скачать 3 исполняемых файла и добавить папку в PATH:
1. [Minikube](https://kubernetes.io/ru/docs/tasks/tools/install-minikube/);
2. [kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/);
3. [helm](https://github.com/helm/helm/releases/latest).

Также, потребуется установить [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

## Запуск Minikube

Запускаем Minikube:
```
minikube start --driver=virtualbox --no-vtx-check
```

## Локальный Image Docker

Самое простое - загрузить в кэш:

```
minikube cache add django_app:latest
```

Но можно ещё собрать в кубе:

```cmd
minikube -p minikube docker-env --shell powershell | Invoke-Expression
```

```bash
eval $(minikube docker-env)
```

```
docker build -t django_app C:\Users\dvmn\k8s\backend_main_django
```

## Секреты
Для создания секрета на кластере Minikube можно использовать .env файл с данными.
Пример содержания .env:
```
SECRET_KEY=DJANGO-SECRET-KEY
DATABASE_URL=postgres://POSTGRES_USER:POSTGRES_PASSWORD_@postgresql.default.svc.cluster.local:5432/POSTGRES
ALLOWED_HOSTS=192.168.1.1,127.0.0.1,localhost
```

Адрес на бд в кластере (установка далее) будет:
```
postgresql.default.svc.cluster.local:5432
```

Далее выполняем команду в директории с файлом:
```
kubectl create secret generic django-secrets --from-env-file=.env
```


## Установка Postgresql
Отредактируйте postgres-values.yaml, указав данные из секретов ранее по примеру.
```
helm install postgresql oci://registry-1.docker.io/bitnamicharts/postgresql -f postgres-values.yaml
```

Если ошибка - команды для удаления:
```
helm uninstall postgresql
kubectl delete pvc -l app.kubernetes.io/instance=postgresql
```

## Запуск

Поднимаем Django:
```
kubectl apply -f deploy.yaml
```

Проверить поды:
```
kubectl get pods
```

Поднимаем сервис:
```
kubectl apply -f service.yaml
```

Выполнить миграцию:
```
kubectl apply -f job-migrate.yaml
```


## Добавление Ingress

Устанавливаем ingress, например, Contour:
```
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```
Изменяем конфиг-файл ingress-ver1.yaml под Ваши предпочтения.

Добавляем ingress
```
kubectl apply -f ingress.yaml
```

Для теста внесены изменения в hosts для локального тестирования:
```
192.168.59.100 star-burger.test
```

## Запуск удаления сессий Django
В манифесте описано создание cronjob каждый 1ый день месяца.
Добавление CronJob:
```
kubectl apply -f cronjob-clearsessions.yaml
```

Для запуска вне расписания используйте, например:
```
kubectl create job --from=cronjob/django-clearsessions django-clearsessions-manual
```

Если job не выполняется - посмотрите логи pod, созданного job:
```
kubectl get pods
```
Вывод, например, такой:
```
django-788cf55d49-kbk4r             1/1     Running        0          45s
django-clearsessions-manual-mmlwh   0/1     ErrImagePull   0          42s
```
Смотрим так:
```
kubectl logs django-clearsessions-manual-mmlwh
```

Шпаргалка на все используемые в проекте для запуска и удаления команды:
Старт.

Миникуб
```
minikube start --driver=virtualbox --no-vtx-check
```
Кидаем образ из докера
```
minikube cache add django_app:latest
```
Секреты
```
kubectl create secret generic django-secrets --from-env-file=.env
```
Postgresql
```
helm install postgresql oci://registry-1.docker.io/bitnamicharts/postgresql -f postgres-values.yaml
```
Django
```
kubectl apply -f deploy.yaml
```
Django - миграция
```
kubectl apply -f job-migrate.yaml
```
Сервис
```
kubectl apply -f service.yaml
```
CronJob на чистку сессий 
```
kubectl apply -f cronjob-clearsessions.yaml
```
Ingress
```
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```
Пример локального теста Ingress
```
kubectl apply -f ingress.yaml
```
```
minikube ip
```
Открываем hosts и вписываем:
(ip из команды выше) star-burger.test

Очистка (немного хаотично):
```
kubectl delete -f deploy.yaml
```
```
kubectl delete -f  job-migrate.yaml
```
```
kubectl delete -f service.yaml
```
```
kubectl delete -f cronjob-clearsessions.yaml
```
```
kubectl delete -f ingress.yaml
```
```
helm uninstall postgresql
```
```
kubectl delete pvc -l app.kubernetes.io/instance=postgresql
```
```
kubectl delete secrets django-secrets
```
```
minikube delete
```