
# Домашнее задание к занятию "11.02 Микросервисы: принципы"

Вы работаете в крупной компанию, которая строит систему на основе микросервисной архитектуры.
Вам как DevOps специалисту необходимо выдвинуть предложение по организации инфраструктуры, для разработки и эксплуатации.

## Задача 1: API Gateway 

Предложите решение для обеспечения реализации API Gateway. Составьте сравнительную таблицу возможностей различных программных решений. На основе таблицы сделайте выбор решения.

Решение должно соответствовать следующим требованиям:
- Маршрутизация запросов к нужному сервису на основе конфигурации
- Возможность проверки аутентификационной информации в запросах
- Обеспечение терминации HTTPS

Обоснуйте свой выбор.

### Решение
  
Требования | NGINX | HAProxy | Varnish 
--- | --- | --- | --- |
Маршрутизация запросов к нужному сервису на основе конфигурации | Да | Да | Да
Возможность проверки аутентификационной информации в запросах | Да | Да | Нет
Обеспечение терминации HTTPS | Да | Да | Нет

Можно остановиться на NGINX:
- Популярность - больше специалистов работает с NGINX
- Балансировка одна из возможностей NGINX, можно использовать остальной его фунционал, например в качестве полноценного веб-сервера.

---

## Задача 2: Брокер сообщений

Составьте таблицу возможностей различных брокеров сообщений. На основе таблицы сделайте обоснованный выбор решения.

Решение должно соответствовать следующим требованиям:
- Поддержка кластеризации для обеспечения надежности
- Хранение сообщений на диске в процессе доставки
- Высокая скорость работы
- Поддержка различных форматов сообщений
- Разделение прав доступа к различным потокам сообщений
- Протота эксплуатации

Обоснуйте свой выбор.  

### Решение

Требования | Apache Kafka | Apache ActiveMQ | RabbitMQ
--- | --- | --- | --- |
Поддержка кластеризации для обеспечения надежности | Да | Да | Да
Хранение сообщений на диске в процессе доставки| Да | Да | Да
Высокая скорость работы | Да+ | Нет (в сравнении с kafka) | Нет (в сравнении с kafka) 
Поддержка различных форматов сообщений | Да | Да | Да
Разделение прав доступа к различным потокам сообщений | Да | Да | Да
Протота эксплуатации | Нет | Нет | Да
  
Выбор зависит от выбора способа обмена сообщениями.
- Если неообходима классическая очередь сообщений - можно выбрать RabbitMQ (хотя в RabbitMQ можно реализовать и схему публикация-подписка)
    - Паблишеры отправляют сообщения на exchange’и
    - Получатели поддерживают постоянные TCP-соединения с RabbitMQ и объявляют, какую очередь(-и) они получают
- Если нужна модель публикация-подписка, можно выбрать Kafka

---
<details>

  <summary>Spoiler warning</summary>

  

  Spoiler text. Note that it's important to have a space after the summary tag. You should be able to write any markdown you want inside the `<details>` tag... just make sure you close `<details>` afterward.

  

  ```javascript

  console.log("I'm a code block!");

  ```

  

</details>
## Задача 3: API Gateway * (необязательная)

### Есть три сервиса:

**minio**
- Хранит загруженные файлы в бакете images
- S3 протокол

**uploader**
- Принимает файл, если он картинка сжимает и загружает его в minio
- POST /v1/upload

**security**
- Регистрация пользователя POST /v1/user
- Получение информации о пользователе GET /v1/user
- Логин пользователя POST /v1/token
- Проверка токена GET /v1/token/validation

### Необходимо воспользоваться любым балансировщиком и сделать API Gateway:

**POST /v1/register**
- Анонимный доступ.
- Запрос направляется в сервис security POST /v1/user

**POST /v1/token**
- Анонимный доступ.
- Запрос направляется в сервис security POST /v1/token

**GET /v1/user**
- Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/
- Запрос направляется в сервис security GET /v1/user

**POST /v1/upload**
- Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/
- Запрос направляется в сервис uploader POST /v1/upload

**GET /v1/user/{image}**
- Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/
- Запрос направляется в сервис minio  GET /images/{image}

### Ожидаемый результат

Результатом выполнения задачи должен быть docker compose файл запустив который можно локально выполнить следующие команды с успешным результатом.
Предполагается что для реализации API Gateway будет написан конфиг для NGinx или другого балансировщика нагрузки который будет запущен как сервис через docker-compose и будет обеспечивать балансировку и проверку аутентификации входящих запросов.
Авторизаци
curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/token

**Загрузка файла**

curl -X POST -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I' -H 'Content-Type: octet/stream' --data-binary @yourfilename.jpg http://localhost/upload

**Получение файла**
curl -X GET http://localhost/images/4e6df220-295e-4231-82bc-45e4b1484430.jpg

---

#### [Дополнительные материалы: как запускать, как тестировать, как проверить](https://github.com/netology-code/devkub-homeworks/tree/main/11-microservices-02-principles)  


### Решение

В задании несколько неточностей:
- Чтобы заработало нужно установить новую версию Flask (requirements.txt > Flask==2.0.3).
- **POST /v1/register** отсутствует в server.py
- **GET /v1/user** отсутствует в server.py

**Конфиг nginx:**
https://github.com/rdegtyarev/devkub-11-02/blob/main/task-3/gateway/nginx.conf  

**docker-compose:**  
https://github.com/rdegtyarev/devkub-11-02/blob/main/task-3/docker-compose.yaml

**Примеры команд для проверки:**

Получить токен
```
curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/token
```  

Загрузить изображение
```
$ curl -X POST -H 'Authorization: Bearer <<your token>>' -H 'Content-Type: octet/stream' --data-binary @1.jpg http://localhost/upload
```  

Скачать изображение
```
curl -X GET -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I' http://localhost/images/<<your image>>.jpg > 1.jpg
```
---
