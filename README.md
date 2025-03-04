# Домашнее задание к занятию «Микросервисы: подходы» - Илларионов Дмитрий

Вы работаете в крупной компании, которая строит систему на основе микросервисной архитектуры.
Вам как DevOps-специалисту необходимо выдвинуть предложение по организации инфраструктуры для разработки и эксплуатации.


## Задача 1: Обеспечить разработку

Предложите решение для обеспечения процесса разработки: хранение исходного кода, непрерывная интеграция и непрерывная поставка. 
Решение может состоять из одного или нескольких программных продуктов и должно описывать способы и принципы их взаимодействия.

Решение должно соответствовать следующим требованиям:
- облачная система;
- система контроля версий Git;
- репозиторий на каждый сервис;
- запуск сборки по событию из системы контроля версий;
- запуск сборки по кнопке с указанием параметров;
- возможность привязать настройки к каждой сборке;
- возможность создания шаблонов для различных конфигураций сборок;
- возможность безопасного хранения секретных данных (пароли, ключи доступа);
- несколько конфигураций для сборки из одного репозитория;
- кастомные шаги при сборке;
- собственные докер-образы для сборки проектов;
- возможность развернуть агентов сборки на собственных серверах;
- возможность параллельного запуска нескольких сборок;
- возможность параллельного запуска тестов.

Обоснуйте свой выбор.

### Решение

#### облачная система

Будем использовать яндекс облако, т.к. там много ресурсов, и функциональности, высокая стабильность, отлаженность, можно доверять.
Можно сосредоточиться на бизнес процессах - разработки ПО (системы), и не заморачиваться администрированием "железа", кондиционированием и т.п.
Это выгодно, если относительно не большие ресурсы нужно иметь, и не выгодно для администрирования и хостинга использовать свой собственный дата-центр, с оборудованным помещением, отказоустойчивость, персоналом.

### система контроля версий Git

Будем использовать GitLab-CI т.к. это современное, много-функциональное, удобное решение (инструмент), бесплатный.
Как вариант - можно использовать облачное решение как сервис у яндекс облака, чтобы не тратить время на установку, поддержку, обновление.
Или альтернатива, если есть специалисты - можно на базе чистой ВМ развернуть самостоятельно gitlab-ci, его настроить, например работу по https.
Преимущества - тут мы сами разбираемся в деталях настроек, и можем болше углубиться в возможности. Но, самим придется и обновлять, планировать работы.
Т.е. в составе gitlab-ci есть сервис git, там и будем хранить код. 
Не будем использовать бес githab.com или gitlab.com т.к.:
* сторонние ресурсы, нет гарантии, что они "завтра" останутся доступными, нет доверия.


### репозиторий на каждый сервис

На этапе проектирования - определим из каких сервисов будет состоять система. Каждый сервис - отдельный функционал, который будет работать в отдельном контейнере, соответственно из подготовленного собранного образа.
Т.е. для каждого сервиса будет свой репозиторий с исзходным кодом, и из этого кода и др. артефактов, например из начальных сторонних образов, будут собираться наши образы.


### запуск сборки по событию из системы контроля версий

В каждом репозитории мы настроим файл .gitlab-ci в котором пропишем yaml скрипт, который будет готовить образ.
В файле скрипта можно определить триггеры по которым будем запускать сборку.


### запуск сборки по кнопке с указанием параметров

В дополнение к автоматической сборки, можно запускать сборку вручную.
Можно подготовить отедльные сборки по кнопке.

### возможность привязать настройки к каждой сборке

В GitLab-ci создадим группу. Далее в этой группе создадим резпозитории на каждый сервис. Для группы определим переменные в gitlab-ci.
Через эти переменные можно задавать значения для сборки.
Так же можно использовать разные ветки в git и некоторые параметры сборки указывать в файлах.

Еще вариант - использовать ансибл роли.
Т.е. в отдельном репозитории мы развиваем роль, и храним исходный код.
И этот код (функционал) уже используем в разных других репозиториях для разного окружения, и подкладываем разные файлы inventory в которых прописываем индивидуальные конфигурации среды. Можно в одном репозитории заготовить сразу несколько файлов конфигураций для разных сред, и запускать конкретный файл, нужный в данный момент для данной среды.
Например, на этапе тестов, мы разворачиваем ВМ в тестовом окружении, и запускаем ансибл с файлом инвентори для тестового окружения для тестов. Если все тесты прошли ок, то,потом можно запустить уже ансибл скрипт в штатном окружении - использовать уже другой файл инвентори из этого же репозитория.

### возможность создания шаблонов для различных конфигураций сборок

Один из вариантов - использовать ансибл-роли (описал выше).
Еще вариат  использовать докер образы с входными параметрами. Т.е. необходимые параметры для среды, задавать уже на этапе запуска контейнера, например в docker-compose.yaml файле. При чем сами параметры можно вынести в переменные среды, их определять отдельно (и не править сам yaml файл для разных сред).
Можно использовать .env файлы и их указыавть при запуске docker-compose.yaml или docker file.

### возможность безопасного хранения секретных данных (пароли, ключи доступа)

Есть много вариантов:
Один из вариантов использовать vault on hashicorp - центральное место (сервис) для хранения паролей и секретов.
Многофункциональный, но, минус - нужно иметь сервер, разбираться, настраивать, поддерживать, обновлять, администрировать.

Есть более простые варианты (но менее функцоональные и менее безопасные):
- можно пароли определить в параметрах среды через экспорт переменных. 
- можно пароли и секретные параметры вручную положить в файл на рабочий сервер в специальную папку, доступ к которой будет только у работающего сервиса, или группы - т.е. на уровне файловой системы определить кому нужно давать доступы. Тогда запускаемый сервис  найдет "строки подключения и пароли". Минусы нужно вручную класть файлы.
- удобный вариант - если использовать ansible можно через ansible-vault зашифровать пароли и положить их в исходный код, и безопасно хранить в git. В момент запуска скрипта Ансибл ввести мастер пароль вручную (или можно положить его в файл домашнюю папку пользователя) и тогда ансибл настроит сервис, расшифрует пароли, и подготовит нужные конфиги (и настроит нужный доступ к конфигам).
- еще вариант, если используем docker-swarm - использовать docker secret. Примеры: 
https://habr.com/ru/articles/659813/
https://cloud.k2.tech/blog/about-technologies/docker-secrets/?ysclid=lyqtvi7wlt682499017


### несколько конфигураций для сборки из одного репозитория

Конфигурации можно указывать в разных в файлах инвентори (ансибл), и использовать их для разных сборок.
или в разных файлах определять разные значения для переменных окружения. Т.е. использовать разные файлы.

### кастомные шаги при сборке

Все нужные шаги будем определять в .gitlab-ci файле в скрипте.


### собственные докер-образы для сборки проектов

Будем использовать начальные образы из публичных репозиториев, но, хранить их будем у себя например в nexus sonatype сервисе - репозитории в docker registry . Это полезно что бы былть независимым от Интернета, все что нужно хранить у себя, и что бы быстрее была бы сборка.
В дополнение к начальным образам будем делать уже свои кастомные слои при сборке своих образов.

### возможность развернуть агентов сборки на собственных серверах

Можно использовать gilab-runner, установленный на своих отдельных ВМ. 

### возможность параллельного запуска нескольких сборок

Можно запускать в параллель несколько сборок, если будем использовать разные раннеры.

### возможность параллельного запуска тестов.

Для разных сервисов использовать разные раннеры, и можно в паралель запускать тесты.

---


## Задача 2: Логи

Предложите решение для обеспечения сбора и анализа логов сервисов в микросервисной архитектуре.
Решение может состоять из одного или нескольких программных продуктов и должно описывать способы и принципы их взаимодействия.

Решение должно соответствовать следующим требованиям:
- сбор логов в центральное хранилище со всех хостов, обслуживающих систему;
- минимальные требования к приложениям, сбор логов из stdout;
- гарантированная доставка логов до центрального хранилища;
- обеспечение поиска и фильтрации по записям логов;
- обеспечение пользовательского интерфейса с возможностью предоставления доступа разработчикам для поиска по записям логов;
- возможность дать ссылку на сохранённый поиск по записям логов.

Обоснуйте свой выбор.


### Решение

Обязательно должно быть централизованный сбор и анализ логов со всех микросервисов в одном месте. И что бы можно было бы и алерты настроить, зонтичный мониторинг.

Есть несколько вариантов решения:

1. ELK - стек ElasticSearch + LogStash + Kibana. Широко распростарнено. Много функционально мощное решение - легко искать логи поиском, или с помощью фильтров, строить графики, дашборды, диаграммы. Но производитель, не работает с Россией, есть некоторые блокировки для скачки, и, важно - есть ограничения по лицензированию - есть обязанность (согласно лцензированию) открывать весь свой исходный код если используем ELK (примерно). Подробнее см. https://wp-club.ru/news/elastic-menyaet-litsenzii-elasticsearch-i-kibana-c-apache-2-0-na-sspl/ .
Плюс многие опции ELK могут быть платными. Например если хотим делать репликацию Elastic Serch то нужно покупать лицензию (на сколько понял).
Но, если решение внутри организации, то, оцениваю  - риск минимален, и допустимо использовать ELK.

2. Альтернатива - OpenSearch 

OpenSearch - это форк от версий ELK, когда ELK был  еще полностью откртым ПО. И, с тех пор сообщество в параллель развивает открытое ПО - OpenSearch.

https://opensearch.org/downloads.html 

3. Еще вариант исползовать Vector + clickhouse открытое ПО от Яндекса.
Похожее ПО но от яндекса.

4. Еще есть вариант стек EFK - Logstash/Beats заменяется на Fluentd или Fluent Bit для создания стека EFK.

5. Еще есть вариант Grafana Loki - меньше функционала чем у ELK .

6. Еще есть Graylog - меньше функционала и удобства чем у ELK.

По моей оценки самый удобный инструмент - смотреть логи через kibana. Доставлять логи можно чере logstash или fluntd. 

При этом важно собирать логи еще со всех контейнеров, например если собирать через Filebit то, можно так сконфигурировать сбор логов из всех контейнеров, размещенных  на сервере: 

```
filebeat.inputs:
  - type: container
    paths:
      - '/var/lib/docker/containers/*/*.log'

processors:
  - add_docker_metadata:
      host: "unix:///var/run/docker.sock"

  - decode_json_fields:
      fields: ["message"]
      target: "json"
      overwrite_keys: true
```

Если ПО система внутри закрытого контура - внутри компании для внутреннего пользования, то, считаю риски минимальны - можно использовать ELK.

Если открытая система - то можно поробовать OpenSearch или Vector + ClickHause.

 
## Задача 3: Мониторинг

Предложите решение для обеспечения сбора и анализа состояния хостов и сервисов в микросервисной архитектуре.
Решение может состоять из одного или нескольких программных продуктов и должно описывать способы и принципы их взаимодействия.

Решение должно соответствовать следующим требованиям:
- сбор метрик со всех хостов, обслуживающих систему;
- сбор метрик состояния ресурсов хостов: CPU, RAM, HDD, Network;
- сбор метрик потребляемых ресурсов для каждого сервиса: CPU, RAM, HDD, Network;
- сбор метрик, специфичных для каждого сервиса;
- пользовательский интерфейс с возможностью делать запросы и агрегировать информацию;
- пользовательский интерфейс с возможностью настраивать различные панели для отслеживания состояния системы.

Обоснуйте свой выбор.

### Решенеие по мониторингу

Для мониторинга статичных хостов - серверов лучше использовать Zabbix - много функциональное ПО. Есть множество шаблонов готовых и решений, например, есть решения и для мониторинга psql. Но Zabbix не очень удобно использовать для мониторинга динамических сред - контейнеров. 
Для мониторинга динамических сред - контейнеров в кластере лучше использовать Prometheus + Grafana.

Для заббикса - ставим на сервера zabbix agent или zabbix aget 2 если нужно еще мониторинг - контейнеров. 

Для Prometheus ставим експортеры для съема данных - параметра хоста. 
Если используем докер, то, в этом ПО уже есть интерфейс для съема данных о контейнерах прометеусом.

Используем графану и можно использовать готовые шаблоны, для создания дашбордов.
можно создавать свои дашборды, и делать свои запросы данных.
 

## Задача 4: Логи * (необязательная)

Продолжить работу по задаче API Gateway: сервисы, используемые в задаче, пишут логи в stdout. 

Добавить в систему сервисы для сбора логов Vector + ElasticSearch + Kibana со всех сервисов, обеспечивающих работу API.

### Результат выполнения: 

docker compose файл, запустив который можно перейти по адресу http://localhost:8081, по которому доступна Kibana.
Логин в Kibana должен быть admin, пароль qwerty123456.


## Задача 5: Мониторинг * (необязательная)

Продолжить работу по задаче API Gateway: сервисы, используемые в задаче, предоставляют набор метрик в формате prometheus:

- сервис security по адресу /metrics,
- сервис uploader по адресу /metrics,
- сервис storage (minio) по адресу /minio/v2/metrics/cluster.

Добавить в систему сервисы для сбора метрик (Prometheus и Grafana) со всех сервисов, обеспечивающих работу API.
Построить в Graphana dashboard, показывающий распределение запросов по сервисам.

### Результат выполнения: 

docker compose файл, запустив который можно перейти по адресу http://localhost:8081, по которому доступна Grafana с настроенным Dashboard.
Логин в Grafana должен быть admin, пароль qwerty123456.

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
