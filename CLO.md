Spring Cloud. Введение
Что такое Spring Cloud?

Spring Cloud — это набор проектов и библиотек, предназначенных для создания cloud-native приложений, прежде всего микросервисных.

Важно понимать одну вещь:

Spring Cloud ≠ отдельный фреймворк.

Это огромное семейство модулей, каждый из которых решает какую-то проблему, возникающую при разработке распределённых приложений.

Например:

Spring Cloud Config
Spring Cloud Gateway
Spring Cloud OpenFeign
Spring Cloud LoadBalancer
Spring Cloud Circuit Breaker
Spring Cloud Stream
Spring Cloud Function
Spring Cloud Kubernetes

Все они работают поверх Spring Boot.

Что означает Cloud Native?

Это один из самых важных терминов.

Cloud Native (дословно — "рожденный для облака") означает, что приложение проектируется так, чтобы работать в современных распределённых средах.

Это не означает, что приложение обязательно должно работать в AWS или Azure.

Cloud Native приложение может работать:

на вашем ноутбуке;
на собственном сервере компании;
в Kubernetes;
в Docker;
в AWS;
в Azure;
в Google Cloud.

То есть "Cloud Native" — это не место запуска.

Это способ проектирования приложения.

Основные идеи Cloud Native

Такие приложения обычно обладают следующими свойствами:

состоят из множества сервисов;
сервисы могут запускаться в нескольких экземплярах;
экземпляры могут постоянно появляться и исчезать;
сервисы должны автоматически находить друг друга;
конфигурация хранится отдельно от кода;
приложение легко масштабируется;
отказ одного сервиса не должен "ронять" всю систему.

Именно эти проблемы Spring Cloud помогает решить.

Почему обычного Spring Boot недостаточно?

Представим интернет-магазин.

Есть сервисы:

User Service
Product Service
Order Service
Payment Service
Notification Service

Каждый сервис — отдельное Spring Boot приложение.

Возникает множество вопросов.

Например...

Как сервисы находят друг друга?

Order Service должен вызвать Payment Service.

Но где он находится?

http://192.168.0.10:8080

?

Через минуту контейнер перезапустился.

Теперь адрес другой:

http://172.18.0.4:8080

Order Service уже ничего не знает.

Это называется проблемой Service Discovery.

Spring Cloud умеет её решать.

Где хранить настройки?

Допустим есть 30 сервисов.

Во всех есть:

spring.datasource.url

или

jwt.secret

или

api.key

Если завтра пароль изменится?

Нужно менять настройки в 30 приложениях.

Неудобно.

Поэтому появляется централизованное хранение конфигурации.

Spring Cloud Config решает именно эту задачу.

Что делать, если один сервис умер?

Допустим:

Order Service
|
V
Payment Service

Payment Service перестал отвечать.

Если ничего не сделать, то:

Order
↓
ожидает...
↓
ожидает...
↓
Timeout
↓
новый запрос
↓
снова Timeout

Через некоторое время потоков не останется вообще.

Упадёт уже Order Service.

Чтобы этого не происходило, используют Circuit Breaker.

Spring Cloud предоставляет поддержку этого механизма.

Как распределять нагрузку?

Допустим работают три экземпляра Payment Service.

Payment #1

Payment #2

Payment #3

Order Service должен отправлять запросы не всегда в один сервер.

Нужно распределять нагрузку.

Это называется Load Balancing.

Spring Cloud умеет делать это автоматически.

Как направлять запросы?

Допустим клиент вызывает

GET /orders

Куда этот запрос должен попасть?

В Order Service.

А

GET /users

В User Service.

Этим занимается API Gateway.

Spring Cloud Gateway предоставляет такую возможность.

Что означает "Spring Cloud убирает Boilerplate"?

Boilerplate — это повторяющийся код.

Без Spring Cloud вам пришлось бы самим писать:

регистрацию сервисов;
поиск сервисов;
балансировку;
обработку ошибок;
повторные запросы;
маршрутизацию;
загрузку конфигурации.

Spring Cloud делает большую часть этого автоматически.

Например, вместо собственного HTTP-клиента с ручным указанием адреса:

RestTemplate restTemplate = new RestTemplate();

restTemplate.getForObject(
"http://192.168.0.15:8080/users/1",
User.class
);

можно написать:

@FeignClient(name = "user-service")
public interface UserClient {

    @GetMapping("/users/{id}")
    User getUser(@PathVariable Long id);
}

Spring Cloud сам найдёт нужный сервис.

Что значит "Proxy behavior from the IoC"?

Лектор упоминает, что Spring Cloud использует proxy behavior from the IoC container.

Это означает следующее.

Когда вы пишете:

@FeignClient

никакого объекта не существует.

Spring создаёт его сам.

Получается примерно так:

Вы вызываете

userClient.getUser(1)

↓

Proxy

↓

ищет сервис

↓

выбирает экземпляр

↓

отправляет HTTP-запрос

↓

возвращает объект

То есть вы работаете с обычным Java-интерфейсом.

Всю сложную работу делает Spring.

Централизованная, но распределённая конфигурация

Лектор говорит интересную фразу:

Centralized but distributed configuration management.

Сначала она кажется противоречивой.

Почему "централизованная", но одновременно "распределённая"?

Представим.

Есть Git-репозиторий:

config-repository

application.yml

user-service.yml

payment-service.yml

order-service.yml

Все настройки лежат здесь.

Это централизованное хранилище.

Но когда сервис запускается, он получает только свои настройки.

User Service
↑
|
Config Server
|
↓
Order Service
|
↓
Payment Service

Каждый сервис хранит свою локальную копию конфигурации.

Поэтому конфигурация:

хранится централизованно;
используется распределённо.
Service Discovery

Это один из самых известных компонентов Spring Cloud.

Допустим имеются сервисы.

Order

User

Payment

Notification

Каждый запускается в Docker.

IP постоянно меняются.

Вместо хранения IP используется реестр сервисов.

Registry

User Service

→ 10.0.0.5

→ 10.0.0.6

Payment Service

→ 10.0.0.9

→ 10.0.0.10

Order Service спрашивает:

Где сейчас Payment Service?

Реестр отвечает.

Circuit Breaker

Идея похожа на автоматический выключатель электричества.

Если сервис отвечает ошибками:

❌

❌

❌

❌

Circuit Breaker говорит:

Я больше не буду даже пытаться обращаться к нему.

И сразу возвращает ошибку или запасной ответ.

Через некоторое время он попробует снова.

Так защищается вся система.

Load Balancer

Есть несколько экземпляров сервиса.

Payment #1

Payment #2

Payment #3

Запросы распределяются:

1

↓

#1

2

↓

#2

3

↓

#3

4

↓

#1

Так нагрузка делится между всеми экземплярами.

Routing (маршрутизация)

Вместо того чтобы клиент знал адрес каждого сервиса, используется Gateway.

Client

↓

Gateway

↓

User Service

↓

Order Service

↓

Payment Service

Клиент общается только с Gateway.

Telemetry

Telemetry — это сбор информации о работе приложения.

Например:

сколько времени выполняется запрос;
какие сервисы он прошёл;
где произошла ошибка;
сколько запросов в секунду;
какие сервисы самые медленные.

Раньше для этого использовался Spring Cloud Sleuth, который автоматически добавлял идентификаторы трассировки (Trace ID, Span ID) и позволял отслеживать путь запроса через несколько микросервисов. В современных версиях Spring Boot и Spring Cloud его роль в основном перешла к Micrometer Tracing, который интегрируется с системами вроде Zipkin или Jaeger.

Spring Cloud Function

Позволяет писать функции, которые можно запускать:

как обычный Spring Bean;
как REST endpoint;
как AWS Lambda;
как Azure Function;
как Google Cloud Function.

Один и тот же код может работать в разных окружениях.

Интеграция с облаками

Spring Cloud умеет работать с:

AWS;
Azure;
Google Cloud;
Kubernetes;
другими платформами.

Для этого существуют специальные Spring Boot Starter'ы.

Например:

spring-cloud-aws

Он автоматически подключает сервисы AWS.

Netflix и Spring Cloud

Лектор говорит, что Netflix сильно повлиял на Spring Cloud.

Почему?

Netflix одной из первых компаний массово использовала микросервисную архитектуру и разработала множество инструментов для решения возникающих проблем. Многие из них были открыты для сообщества и позже интегрированы в Spring Cloud.

Самые известные:

Eureka — Service Discovery.
Ribbon — Load Balancer (сейчас устарел, заменён Spring Cloud LoadBalancer).
Hystrix — Circuit Breaker (устарел, вместо него обычно используют Resilience4j через Spring Cloud Circuit Breaker).
Zuul — API Gateway (в новых проектах чаще используют Spring Cloud Gateway).
Что нужно запомнить

Spring Cloud не заменяет Spring Boot — он расширяет его возможностями для распределённых систем.

Основные задачи Spring Cloud:

централизованное управление конфигурацией (Spring Cloud Config);
обнаружение сервисов (Service Discovery);
балансировка нагрузки (Load Balancer);
отказоустойчивость (Circuit Breaker);
маршрутизация запросов (Gateway);
декларативные HTTP-клиенты (OpenFeign);
распределённая трассировка и телеметрия (Micrometer Tracing, ранее Spring Cloud Sleuth);
интеграция с облачными платформами и serverless-функциями.
Что важно понять до изучения Spring Cloud

Если проводить аналогию с изучением Spring Security, то здесь тоже есть фундаментальная идея:

Spring Boot отвечает за создание одного приложения.
Spring Cloud отвечает за взаимодействие множества Spring Boot-приложений между собой.

Поэтому большинство технологий Spring Cloud начинают приносить пользу тогда, когда у вас появляется несколько микросервисов, которые должны безопасно и надёжно общаться друг с другом. В следующих темах вы увидите, как именно решаются эти задачи: сначала конфигурация, затем обнаружение сервисов, балансировка нагрузки, шлюзы, отказоустойчивость и другие компоненты распределённой архитектуры.

Externalized Configuration (Внешняя конфигурация)
Что вообще такое конфигурация?

Конфигурация — это данные, которые влияют на работу программы, но не являются частью её бизнес-логики.

Другими словами:

Код отвечает КАК работать, а конфигурация отвечает С ЧЕМ работать.

Например.

Есть сервис оплаты.

В коде написано:

paymentClient.send(order);

Но куда отправлять запрос?

http://payment-service

или

http://payment-stage

или

http://payment-dev

Это уже конфигурация.

Что такое Externalized Configuration?

Externalized Configuration означает:

Конфигурация хранится вне приложения, а не "зашита" в код.

То есть вместо этого:

String url = "https://prod.payment.com";

мы пишем

payment:
url: https://prod.payment.com

а затем читаем

@Value("${payment.url}")
private String url;

или

@ConfigurationProperties(prefix = "payment")
public class PaymentProperties {
private String url;
}

Теперь код вообще не знает, какой URL использовать.

Он просто берёт его из конфигурации.

Почему нельзя просто написать URL в коде?

Представь.

У тебя есть приложение.

Оно работает:

локально;
на тестовом сервере;
на staging;
в production.

Во всех случаях сервис оплаты находится по разному адресу.

Например

Локально

http://localhost:8085

DEV

http://payment-dev.company.com

STAGE

http://payment-stage.company.com

PROD

https://payment.company.com

Если URL находится в коде, то каждый раз придётся менять код и пересобирать приложение.

Это очень плохая практика.

Что обычно является конфигурацией?

Лектор приводит хорошие примеры.

Разберём каждый.

1. URL других сервисов

Например

user-service:
url: http://user-service

или

payment:
url: https://payment.company.com

Почему?

Потому что завтра сервис может переехать.

Код менять не придётся.

2. Username и Password

Например

spring:
datasource:
username: admin
password: secret

На DEV одна база.

На PROD совершенно другая.

Пароли отличаются.

Код одинаковый.

3. Batch Size

Например

Есть обработка миллионов файлов.

В production выгодно обрабатывать

10000

файлов за раз.

На компьютере разработчика столько памяти нет.

Поэтому

batch:
size: 50

В production

batch:
size: 10000

Код один и тот же.

4. Таймауты

Например

http:
timeout: 3000

или

http:
timeout: 10000
5. Email

Например

DEV

test@company.com

Production

noreply@company.com
6. Encryption Keys

Например

JWT Secret

jwt:
secret: ...

или сертификаты.

Простое правило

Практически всегда можно пользоваться правилом:

Если значение может отличаться между окружениями — это конфигурация.

Например

✔ URL

✔ пароль

✔ порт

✔ размер пула потоков

✔ размер batch

✔ API Key

✔ email отправителя

✔ логирование

✔ feature flags

Но

price * quantity

не является конфигурацией.

Это бизнес-логика.

Почему Externalized Configuration настолько важна?

Вот здесь начинается самое интересное.

Лектор говорит:

Это позволяет запускать один и тот же код где угодно.

Это главный принцип.

Представим.

Есть приложение.

Сегодня оно работает

Docker

Завтра

Kubernetes

Через месяц

AWS

Потом

Azure

Код вообще не меняется.

Меняется только конфигурация.

Код становится переносимым (Portable)

Лектор использует слово

Portable Application

Это означает

Приложение можно взять и перенести в любую среду.

Например

Laptop

↓

Docker

↓

Kubernetes

↓

AWS

↓

Bare Metal Server

Код не меняется.

Меняется только

application.yml

или переменные окружения.

Не нужно писать if'ы

Очень распространённая ошибка новичков.

Например

if (environment.equals("DEV")) {
url = "...";
}
else if (environment.equals("TEST")) {
url = "...";
}
else if (environment.equals("PROD")) {
url = "...";
}

Так делать не стоит.

Spring уже умеет работать с профилями и внешней конфигурацией.

Лучше написать

payment.url=...

И всё.

Почему не хватает Environment Variables?

Возникает закономерный вопрос.

Linux уже умеет хранить

DATABASE_URL
JWT_SECRET
API_KEY

Зачем тогда Spring Cloud Config?

Лектор отвечает именно на этот вопрос.

Причина №1. Version Control

Это, наверное, самый большой плюс.

Конфигурация хранится в Git.

Например

config-repository

application.yml

order-service.yml

payment-service.yml

user-service.yml

Теперь изменения выглядят как обычный Git.

Например

commit

Изменён timeout

или

commit

Изменён URL

Можно посмотреть

кто изменил;
когда;
зачем;
вернуть старую версию.

С переменными окружения так не получится.

Pull Request

Лектор говорит

можно требовать PR

Представим.

Кто-то хочет изменить

jwt.secret

или

payment.url

Он делает

Pull Request

Другой разработчик проверяет изменения.

После этого они попадают в production.

То есть конфигурация проходит тот же процесс проверки, что и код.

Branches

Например

main

для Production

develop

для DEV

release

для Stage

Каждая ветка хранит собственную конфигурацию.

Централизованное управление

Вот это уже главная идея Spring Cloud Config.

Представь 30 микросервисов.

У каждого есть

application.yml

Получается

Order

application.yml

Payment

application.yml

User

application.yml

Email

application.yml

Если нужно поменять

timeout

во всех сервисах?

Придётся открыть 30 файлов.

Это неудобно.

Spring Cloud Config предлагает другое решение.

Есть отдельный сервис.

               Config Server
                     |
      -------------------------------
      |      |      |      |       |
User   Order   Payment Email Inventory

Все приложения получают конфигурацию из одного места.

Это и называется централизованное управление конфигурацией.

Как работает Spring Cloud Config Server?

На самом деле довольно просто.

Шаг 1

Есть Git.

config-repository

В нём лежат YAML-файлы.

Шаг 2

Есть Config Server.

Spring Boot Application

Он подключается к Git.

При запуске читает все настройки.

Шаг 3

Любой сервис обращается к Config Server.

Например

GET

/order-service/default

Config Server отвечает

payment.url=...

jwt.secret=...

timeout=...
Шаг 4

Order Service запускается уже с этими настройками.

То есть вместо чтения локального application.yml он получает конфигурацию от Config Server (обычно на этапе старта приложения).

Почему это удобно?

Представим.

Поменялся URL Payment Service.

Без Config Server:

Order

меняем

↓

Git

↓

Commit

↓

Build

↓

Deploy

С Config Server:

Git

↓

Commit

↓

Config Server

↓

Order Service

Никаких изменений кода.

Поддержка Branch и Profiles

Лектор говорит

поддерживаются branches и profiles.

Что это означает?

Например

Git

main

develop

release

В каждом хранится собственная конфигурация.

Кроме того,

application-dev.yml

application-stage.yml

application-prod.yml

То есть можно комбинировать

ветка

+

Spring Profile

и получать нужную конфигурацию.

Где Spring Cloud Config особенно полезен?

Лектор приводит пример с Kubernetes.

В Kubernetes обычно используются:

ConfigMap;
Secret;
Helm values.

Без Config Server конфигурация может оказаться разбросана по разным Helm-чартам и репозиториям.

Config Server позволяет хранить её централизованно в одном Git-репозитории и раздавать всем приложениям. Это упрощает управление, особенно если микросервисов много.

При этом Spring Cloud Config не привязан к Kubernetes. Он так же полезен и для приложений, работающих на обычных виртуальных машинах, физических серверах или в других облачных средах.

Итоги

Externalized Configuration — это принцип хранения конфигурации отдельно от кода.

Главные преимущества:

один и тот же код работает в любых окружениях (DEV, TEST, STAGE, PROD);
не нужно менять и пересобирать приложение при изменении параметров;
исчезают if по типу окружения;
конфигурацию можно версионировать в Git, использовать Pull Request и откатывать изменения;
появляется единая точка управления настройками всех микросервисов;
приложение становится переносимым (portable) и проще в сопровождении.

Связь с тем, что ты уже изучал: application.properties, application.yml, @Value, @ConfigurationProperties, Environment и Spring Profiles — это механизмы получения конфигурации внутри одного приложения. Spring Cloud Config поднимает эту идею на уровень всей системы: десятки или сотни сервисов получают свои настройки из единого центра, сохраняя при этом независимость кода и гибкость развертывания.

Setting up Spring Cloud Config Server
Что мы хотим получить?

В конце у нас должна получиться такая схема:

                    Git Repository
              (хранит все настройки)
                       │
                       │
             читает конфигурацию
                       │
                       ▼
            Spring Cloud Config Server
                   (порт 8888)
                       │
      ┌────────────────┼─────────────────┐
      │                │                 │
      ▼                ▼                 ▼
Order Service   User Service   Payment Service

То есть:

Настройки лежат в Git.
Config Server читает Git.
Остальные сервисы получают настройки через Config Server.
Шаг 1. Создаём Git-репозиторий

Лектор выполняет

mkdir ~/etc
cd ~/etc
git init

Получается обычный Git-репозиторий.

etc/
├── .git

Зачем Git?

Потому что Spring Cloud Config умеет работать именно с Git.

Преимущества:

история изменений;
откат изменений;
Pull Request;
разные ветки;
удобная совместная работа.

По сути Git становится "базой данных" для конфигурации.

Шаг 2. Создаём файл конфигурации

Лектор создаёт

room-reservation-service.properties

Внутри

guest.service.url=http://localhost:8081
reservation.service.url=http://localhost:8082
room.service.url=http://localhost:8083

Что означают эти параметры?

Допустим существует сервис бронирования комнат.

Он общается ещё с тремя сервисами.

Reservation Service
│
├────────► Guest Service
│
├────────► Room Service
│
└────────► Reservation DB

Вместо

String url = "http://localhost:8081";

адрес хранится в конфигурации.

Почему именно имя файла такое?

Очень важный момент.

Spring Cloud Config использует соглашение об именовании файлов.

Например

order-service.properties

относится к приложению

spring.application.name=order-service

Если приложение называется

spring.application.name=room-reservation-service

то оно автоматически будет искать

room-reservation-service.properties

или

room-reservation-service.yml
Шаг 3. Коммитим изменения
git add .
git commit -m "Adding config"

Теперь Config Server сможет читать этот Git.

Шаг 4. Создаём Spring Boot приложение

Создаётся обычное Spring Boot приложение.

Ничего необычного.

Единственная зависимость

Лектор выбирает

Spring Cloud Config Server

Она добавляет зависимость

spring-cloud-config-server

Именно она превращает обычный Spring Boot в Config Server.

Шаг 5. @EnableConfigServer

Самая важная аннотация.

@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }

}

Что делает эта аннотация?

Она включает всю инфраструктуру Config Server.

После её добавления приложение начинает:

искать Git;
читать конфигурацию;
поднимать REST API;
отдавать настройки клиентам.

То есть один annotation превращает приложение в сервер конфигурации.

Шаг 6. application.properties

Лектор пишет

server.port=8888

Почему именно 8888?

Никакой магии.

Можно поставить

9000

или

8088

или

9999

8888 — просто традиционный порт для Config Server.

Указываем Git Repository

Главная настройка

spring.cloud.config.server.git.uri=file://${HOME}/etc

Она означает

Конфигурация хранится здесь.

Config Server при запуске делает примерно следующее.

Git Repository

↓

открывает

↓

читает все файлы

↓

запоминает

↓

готов отдавать их клиентам
Можно ли использовать удалённый Git?

Да.

Например

spring.cloud.config.server.git.uri=https://github.com/company/config

или

spring.cloud.config.server.git.uri=https://gitlab.company.com/config

Именно так чаще всего делают в реальных проектах.

Что происходит после запуска?

Когда запускается Config Server

Spring Boot

↓

@EnableConfigServer

↓

Git Repository

↓

читает все файлы

↓

строит REST API

То есть никаких настроек он пока никому не отправляет.

Он просто становится сервером.

Как получить конфигурацию?

Лектор открывает браузер

http://localhost:8888/room-reservation-service.properties

И получает

guest.service.url=http://localhost:8081
reservation.service.url=http://localhost:8082
room.service.url=http://localhost:8083

Откуда это появилось?

Не из памяти приложения.

Не из application.properties.

А именно из Git.

То есть цепочка такая

Git

↓

Config Server

↓

REST

↓

Клиент
На самом деле REST API немного богаче

Современный Config Server обычно использует такой шаблон URL:

/{application}/{profile}

Например

GET /order-service/default

или

GET /order-service/dev

или

GET /payment-service/prod

Можно также запрашивать конкретную ветку Git (label):

GET /order-service/prod/main

Ответ приходит в формате JSON и содержит значения свойств, активный профиль, источник конфигурации и другую служебную информацию.

Поддержка URL вида

/room-reservation-service.properties

существует для совместимости и тоже возвращает конфигурацию, но чаще в современных примерах используют REST-эндпоинты с именем приложения и профилем.

Почему Config Server сам является Spring Boot приложением?

Это одна из сильных сторон Spring Cloud.

Не существует какого-то отдельного сервера.

Нет отдельной программы.

Нет отдельного процесса.

Есть обычный Spring Boot.

Ты добавил зависимость

↓

добавил аннотацию

↓

написал две настройки

↓

получил полноценный Config Server.

Что происходит "под капотом"?

Когда клиент запрашивает конфигурацию:

GET /order-service/default

происходит примерно следующее.

                Request
                    │
                    ▼
          Spring MVC Controller
                    │
                    ▼
        Config Server Infrastructure
                    │
                    ▼
             Git Environment
                    │
                    ▼
      Ищет нужные файлы конфигурации
                    │
                    ▼
        Собирает Environment
                    │
                    ▼
         Возвращает JSON клиенту

Важно понимать, что Config Server не «знает» значения заранее. Он читает их из Git-репозитория (локального или удалённого), формирует ответ и отправляет клиенту.

Как клиент использует Config Server?

В этой лекции показан только сервер, но уже можно представить следующий шаг.

Допустим есть Order Service.

При запуске он сначала обращается к Config Server:

Order Service
│
│ GET /order-service/default
▼
Config Server
│
▼
Git Repository

Получает ответ

payment.url=http://payment-service
jwt.secret=...
logging.level.root=INFO

После этого Spring Boot продолжает запускаться уже с полученными настройками, как будто они были в локальном application.yml.

Что важно запомнить
Spring Cloud Config Server — это обычное Spring Boot-приложение с зависимостью spring-cloud-config-server и аннотацией @EnableConfigServer.
Все настройки хранятся в Git-репозитории (локальном или удалённом).
Имя файла конфигурации обычно совпадает со значением spring.application.name клиентского приложения.
При запуске Config Server подключается к Git и предоставляет конфигурацию через REST API.
Клиентские приложения больше не обязаны хранить все настройки локально — они могут получать их централизованно из Config Server.

Практический совет: в современных версиях Spring Cloud многие проекты используют application.yml вместо .properties, а для подключения клиента к Config Server применяется механизм Config Data API (spring.config.import=configserver:), пришедший на смену старому bootstrap.properties. Если будешь писать новый проект на Spring Boot 3.x, ориентируйся именно на этот современный подход.

Consuming Config Server (Получение конфигурации от Config Server)
Что мы сделали в прошлой лекции?

Мы подняли Config Server.

Он выглядит примерно так:

                Git Repository
                      │
                      ▼
            Spring Cloud Config Server
                 localhost:8888

Но пока никто этим сервером не пользуется.

Теперь нужно сделать так, чтобы обычное Spring Boot приложение получало настройки именно от него.

Что такое Config Client?

Любое Spring Boot приложение может стать Config Client.

Например

Order Service

User Service

Payment Service

Inventory Service

Все они могут получать конфигурацию из одного места.

То есть схема становится такой

                   Git
                    │
                    ▼
             Config Server
                    │
      ┌─────────────┼──────────────┐
      ▼             ▼              ▼
Order Service  User Service  Payment Service
Шаг 1. Добавляем Spring Cloud BOM

Вот эта часть вызывает больше всего вопросов.

Лектор пишет

<properties>
    <spring-cloud.version>
        2021.0.3
    </spring-cloud.version>
</properties>

Зачем?

Потому что Spring Cloud состоит из десятков библиотек.

Например

Config
Gateway
Eureka
OpenFeign
Circuit Breaker
LoadBalancer

Все они должны быть совместимы между собой.

Чтобы не писать версии каждой зависимости отдельно, существует BOM.

Что такое BOM?

BOM означает

Bill Of Materials

Можно представить его как список совместимых версий библиотек.

Например

Spring Cloud 2024

↓

Config 4.1

Gateway 4.1

OpenFeign 4.1

LoadBalancer 4.1

Все версии гарантированно работают друг с другом.

dependencyManagement

Далее автор добавляет

<dependencyManagement>
    <dependencies>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>

            <type>pom</type>

            <scope>import</scope>

        </dependency>

    </dependencies>
</dependencyManagement>

Разберём подробно.

Что делает dependencyManagement?

Очень многие думают, что Maven скачивает зависимость.

Нет.

Он делает другое.

Он говорит:

Если где-то встретится зависимость Spring Cloud —
используй версии отсюда.

То есть

<dependency>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>

уже можно писать без версии.

Maven сам её найдёт.

Что означает scope=import?

Это специальный режим только для BOM.

Он говорит Maven

Импортируй список версий из другого pom.xml.

Почему type=pom?

Потому что BOM — это не библиотека.

Это обычный pom-файл с версиями.

Шаг 2. Добавляем Config Client

Теперь добавляется зависимость

<dependency>

    <groupId>org.springframework.cloud</groupId>

    <artifactId>
        spring-cloud-starter-config
    </artifactId>

</dependency>

Что она делает?

Она превращает обычное Spring Boot приложение в клиента Config Server.

После её добавления Spring Boot начинает понимать

Config Server

↓

Git

↓

Remote Configuration
Шаг 3. Удаляем локальные настройки

Раньше было

guest.service.url=...
reservation.service.url=...
room.service.url=...

Теперь всё удаляется.

Почему?

Потому что эти настройки теперь будут храниться в Git.

Что остаётся?

Остаётся всего две настройки.

Первая
spring.application.name=room-reservation-service

Это одна из самых важных настроек Spring Cloud.

Почему?

Потому что именно по ней Config Server понимает

Какие настройки нужно вернуть.

Например

если приложение называется

spring.application.name=user-service

Config Server ищет

user-service.properties

или

user-service.yml

Если

spring.application.name=payment-service

то

payment-service.yml
Почему это имя используется "везде"?

Лектор говорит

spring.application.name используется во всём Spring Cloud.

Это правда.

Например

Service Discovery

User Service

регистрируется именно под этим именем.

Feign

ищет сервис именно по этому имени.

LoadBalancer

балансирует именно это имя.

Config Server

ищет именно этот файл.

Поэтому имя должно быть стабильным.

Вторая настройка
spring.config.import=optional:configserver:http://localhost:8888

Вот это самая интересная строчка.

Она появилась начиная со Spring Boot 2.4.

Раньше использовали

bootstrap.properties

Теперь используют именно

spring.config.import
Что означает configserver?

Spring говорит

Помимо локального application.properties,
ещё сходи на Config Server.

Что означает optional?

Очень хороший вопрос.

Без optional

spring.config.import=configserver:http://localhost:8888

Если Config Server не работает

↓

приложение вообще не запустится.

С optional

optional:configserver:

если сервер недоступен,

Spring просто продолжит запуск (при условии, что необходимых свойств хватает локально или они не обязательны).

Поэтому на практике используют оба варианта — всё зависит от требований проекта.

Что происходит во время запуска?

Вот самая важная часть лекции.

Представим

RoomReservationService

запускается.

Последовательность такая.

Шаг 1

Запускается Spring Boot.

main()

↓

SpringApplication.run()
Шаг 2

Spring читает

spring.application.name

Получает

room-reservation-service
Шаг 3

Видит

spring.config.import=configserver:

Значит

нужно обратиться к Config Server.

Шаг 4

Отправляет HTTP-запрос

Например

GET http://localhost:8888/room-reservation-service/default
Шаг 5

Config Server

↓

читает Git

↓

находит

room-reservation-service.properties

↓

возвращает настройки.

Шаг 6

Spring Boot объединяет

локальные настройки

+

удалённые настройки

Получается Environment.

Шаг 7

Только теперь начинается создание Bean'ов.

То есть

Config Server

↓

Environment

↓

IOC Container

↓

Beans

Именно поэтому

@Value("${guest.service.url}")

уже работает.

Хотя свойства нет в локальном application.properties.

Что означают сообщения в логах?

Лектор показывает

Located environment:

room-reservation-service

profile=default

label=null

version=<git hash>

Разберём каждую строку.

Environment
room-reservation-service

Какой файл был найден.

Profile
default

Какой Spring Profile используется.

Если бы был

dev

искался бы

room-reservation-service-dev.yml

или соответствующий профиль в конфигурации.

Label
null

Label — это ветка Git.

Например

main

или

develop

Если label не указан,

используется ветка по умолчанию.

Version

Очень полезная вещь.

Git hash

Например

d9af351...

Теперь можно узнать

какая именно версия конфигурации используется приложением.

Это невероятно помогает при расследовании ошибок.

Почему приложение продолжило работать?

Лектор говорит

Мы удалили локальную конфигурацию, но приложение продолжило работать.

Почему?

Потому что

раньше

application.properties

↓

Environment

Теперь

Config Server

↓

Environment

Для самого приложения ничего не изменилось.

Оно по-прежнему читает свойства из Environment.

Просто источник этих свойств стал другим.

Откуда RestTemplate получил URL?

Допустим есть

@Value("${guest.service.url}")
private String guestUrl;

Когда Spring создаёт Bean,

он спрашивает

Environment

↓

guest.service.url ?

Environment отвечает

http://localhost:8081

Но сам Environment уже получил это значение от Config Server.

Поэтому RestTemplate продолжает работать без каких-либо изменений в коде.

Главное преимущество

Представим

раньше

Payment URL изменился

↓

меняем application.properties

↓

Commit

↓

Build

↓

Deploy

Теперь

Git Config Repository

↓

меняем URL

↓

Commit

↓

Config Server

↓

клиенты получают новую конфигурацию

Во многих проектах для применения изменений всё ещё требуется перезапуск приложения или использование механизма обновления конфигурации (например, Spring Cloud Bus + Actuator /refresh, в зависимости от версии и архитектуры). Но главное преимущество остаётся: код не меняется, меняется только конфигурация.

Полный жизненный цикл Config Client
Spring Boot запускается
│
▼
Читает spring.application.name
│
▼
Видит spring.config.import=configserver:
│
▼
HTTP-запрос к Config Server
│
▼
Config Server читает Git
│
▼
Возвращает настройки приложения
│
▼
Spring создаёт Environment
│
▼
Создаются все Beans
│
▼
@Value и @ConfigurationProperties
получают значения из Environment
Что нужно запомнить
spring-cloud-starter-config делает Spring Boot приложением Config Client.
spring.application.name определяет, какую конфигурацию запросит клиент у Config Server.
spring.config.import=configserver:... указывает, где находится Config Server.
Во время запуска приложение сначала получает конфигурацию с Config Server, затем формирует Environment, и только после этого начинает создавать бины.
Для кода нет разницы, пришли свойства из локального файла или из Config Server: он всегда работает через Environment. Именно поэтому можно централизованно управлять настройками всех сервисов, не изменяя их исходный код.