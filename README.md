## Секреты
Для создания секрета на кластере Minikube можно использовать .env файл с данными.
Пример содержания .env:
```
SECRET_KEY=DJANGO-SECRET-KEY
DATABASE_URL=postgres://POSTGRES_USER:POSTGRES_PASSWORD_@192.168.1.1:5431/POSTGRES
ALLOWED_HOSTS=192.168.1.1,127.0.0.1,localhost
```

Далее выполняем команду в директории с файлом:
```
kubectl create secret generic django-secrets --from-env-file=.env
```


kubectl apply -f https://projectcontour.io/quickstart/contour.yaml

## Добавление Ingress

Устанавливаем ingress, например, Contour:
```
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```
Изменяем конфиг-файл ingress-ver1.yaml под Ваши предпочтения.

Добавляем ingress
```
kubectl apply -f ingress-ver1.yaml
```

Для теста внесены изменения в hosts для локального тестирования:
```
192.168.59.100 star-burger.test
```