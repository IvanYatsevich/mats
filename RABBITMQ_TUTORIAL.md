# Полное руководство по RabbitMQ: Теория и практика

Содержание:
1. [Что такое RabbitMQ?](#что-такое-rabbitmq)
2. [Основные компоненты](#основные-компоненты)
3. [Как работает RabbitMQ](#как-работает-rabbitmq)
4. [Типы Exchange](#типы-exchange)
5. [Ваш проект - практический пример](#ваш-проект---практический-пример)
6. [Архитектура сообщений](#архитектура-сообщений)
7. [Коды примеров](#коды-примеров)
8. [Бестпрактики и советы](#бестпрактики-и-советы)

---

## Что такое RabbitMQ?

**RabbitMQ** - это брокер сообщений (message broker), написанный на языке Erlang. Это программное обеспечение, которое позволяет разным приложениям взаимодействовать между собой асинхронно через обмен сообщениями.

### Зачем нужен RabbitMQ?

Представьте систему обработки изображений:
- **Без RabbitMQ**: Приложение 1 должно напрямую вызвать Приложение 2, ждать ответа, и если Приложение 2 не работает - система падает или зависает.
- **С RabbitMQ**: Приложение 1 отправляет сообщение в RabbitMQ и забывает о нём. RabbitMQ гарантирует, что сообщение будет доставлено Приложению 2, даже если тот сейчас не работает.

### Основные преимущества:

✅ **Асинхронность** - приложения не ждут друг друга
✅ **Отказоустойчивость** - если потребитель упал, сообщение сохранится
✅ **Масштабируемость** - легко добавить новых потребителей
✅ **Развязывание** - приложения не обязаны знать о друг друге
✅ **Гибкость маршрутизации** - сложные логики доставки сообщений

---

## Основные компоненты

RabbitMQ работает с четырьмя ключевыми компонентами:

### 1. **Producer (Производитель)**

Это приложение, которое **создаёт и отправляет сообщения**.

```
PRODUCER ──→ RabbitMQ ──→ CONSUMER
```

В вашем проекте: `producer` приложение - именно это Producer.
Код: `MessageSender.java`

**Характеристики:**
- Создаёт сообщение
- Отправляет в Exchange
- Не знает о Consumer
- Не нужно ждать подтверждения

---

### 2. **Consumer (Потребитель)**

Это приложение, которое **получает и обрабатывает сообщения**.

```
PRODUCER ──→ RabbitMQ ──→ CONSUMER
```

В вашем проекте: `consumer` приложение - это Consumer.
Код: `MessageListener.java`

**Характеристики:**
- Слушает очередь (Queue)
- Получает сообщение
- Обрабатывает сообщение
- Подтверждает обработку (ACK)

---

### 3. **Exchange (Обменник)**

Exchange - это **почтовое отделение** RabbitMQ.

```
Producer ──→ [Exchange] ──→ Queue ──→ Consumer
```

**Что он делает:**
- Получает сообщения от Producer
- Смотрит на Routing Key сообщения
- Маршрутизирует (отправляет) сообщение в нужную Queue

**Аналогия с почтой:**
- `Exchange` = Почтовое отделение
- `Routing Key` = Код потребителя почтовых услуг
- `Queue` = Почтовый ящик получателя

**В вашем проекте:**

```yaml
messaging-config:
  exchangeName: image.processing.exchange  # ← Это Exchange
  routingKey: thumbnail.create              # ← Это Routing Key
```

Exchange создаётся в конфигурации:

```java
@Bean
public DirectExchange directExchange(MessagingProperties messagingProperties) {
    return new DirectExchange(messagingProperties.getExchangeName());
}
```

---

### 4. **Queue (Очередь)**

Queue - это **ящик с сообщениями**, из которого потребитель берёт сообщения.

```
Producer ──→ Exchange ──→ [Queue] ──→ Consumer
```

**Что произойдёт в очереди:**
- Сообщения приходят в очередь в порядке FIFO (First In, First Out)
- Остаются в памяти (или на диске, если настроено)
- Потребитель берёт сообщения по одному
- После обработки сообщение удаляется

**В вашем проекте:**

```yaml
messaging-config:
  queueName: image.processing.queue  # ← Это Queue
```

Queue создаётся в конфигурации:

```java
@Bean
public Queue queue(MessagingProperties messagingProperties) {
    return new Queue(messagingProperties.getQueueName());
}
```

---

### 5. **Binding (Связь)**

Binding - это **правило маршрутизации**, которое связывает Exchange и Queue.

```
Producer ──→ Exchange ── binding──→ Queue ──→ Consumer
```

**Что он делает:**
- Говорит Exchange: "Когда придёт сообщение с RoutingKey='thumbnail.create', отправь его в Queue 'image.processing.queue'"

**В вашем проекте:**

```java
@Bean
public Binding binding(Queue queue, DirectExchange directExchange, 
                       MessagingProperties messagingProperties) {
    return BindingBuilder.bind(queue)
            .to(directExchange)
            .with(messagingProperties.getRoutingKey());
}
```

---

### 6. **Routing Key (Ключ маршрутизации)**

Routing Key - это **адрес** сообщения, который помогает Exchange определить, в какую Queue отправить сообщение.

**Аналогия:**
- `Routing Key` = Адрес получателя на письме
- `Exchange` = Почтовое отделение, которое читает адрес и отправляет письмо нужному ящику

**В вашем проекте:**

```yaml
messaging-config:
  routingKey: thumbnail.create
```

Producer отправляет с этим ключом:

```java
public void sendMessage(ThumbnailCreationRequest request) {
    rabbitTemplate.convertAndSend(
            messagingProperties.getExchangeName(),    // Exchange
            messagingProperties.getRoutingKey(),      // Routing Key
            request                                    // Сообщение
    );
}
```

---

## Как работает RabbitMQ

Полный процесс от A до Z:

### Шаг за шагом:

```
1. PRODUCER (приложение)
   ↓
   Создаёт объект ThumbnailCreationRequest
   imagePath = "c:\image-demo\images\photo.jpg"
   
2. PRODUCER отправляет сообщение
   ↓
   rabbitTemplate.convertAndSend(
       "image.processing.exchange",  // Exchange
       "thumbnail.create",           // Routing Key
       request                       // Сообщение (сериализуется в JSON)
   )
   
3. RABBITMQ BROKER (сервер)
   ↓
   Получает сообщение на Exchange
   "image.processing.exchange"
   
4. EXCHANGE получает сообщение
   ↓
   Читает Routing Key: "thumbnail.create"
   Смотрит на Binding правила
   Находит, что Binding говорит:
   "RoutingKey='thumbnail.create' → Queue='image.processing.queue'"
   
5. EXCHANGE отправляет сообщение в Queue
   ↓
   Сообщение попадает в очередь
   "image.processing.queue"
   
6. CONSUMER (приложение) слушает Queue
   ↓
   @RabbitListener(queues = "image.processing.queue")
   public void processImage(ThumbnailCreationRequest request) {
       // Обработка сообщения
   }
   
7. RABBITMQ смотрит на очередь
   ↓
   Видит, что есть свободный Consumer
   Отправляет ему сообщение из Queue
   
8. CONSUMER обрабатывает сообщение
   ↓
   Получает ThumbnailCreationRequest
   Создаёт thumbnail из изображения
   Сохраняет результат
   
9. CONSUMER подтверждает (ACK)
   ↓
   "Я обработал сообщение"
   RabbitMQ удаляет сообщение из Queue
```

---

## Типы Exchange

В RabbitMQ есть разные типы Exchange, каждый с собственной логикой маршрутизации.

### 1. **Direct Exchange** (Прямой обменник)

**Логика:** Сообщение отправляется в Queue, чей Binding имеет точно такой же Routing Key.

```
Producer: routingKey = "thumbnail.create"
    ↓
Exchange (Direct): "Кто слушает routingKey='thumbnail.create'?"
    ↓
Binding: "Есть Queue 'image.processing.queue' слушает 'thumbnail.create'"
    ↓
Queue → Consumer
```

**Пример маршрутизации:**

```
routingKey="order.created" ──→ Queue="orders"      ✓
routingKey="order.failed"  ──→ Queue="failed"      ✓
routingKey="payment.done"  ──→ Queue="orders"      ✗ (не совпадает)
```

**Когда использовать:** Для простой 1:1 маршрутизации. В вашем проекте используется именно это.

**В вашем коде:**

```java
@Bean
public DirectExchange directExchange(MessagingProperties messagingProperties) {
    return new DirectExchange(messagingProperties.getExchangeName());
}
```

---

### 2. **Fanout Exchange** (Рассылка)

**Логика:** Сообщение отправляется во ВСЕ Queue, привязанные к этому Exchange (Routing Key игнорируется).

```
Producer отправляет сообщение
    ↓
Exchange (Fanout): "Отправить это ВСЕМ!"
    ↓
Все привязанные Queue получают копию сообщения
    ↓
Все Consumer обрабатывают сообщение
```

**Пример:**

```
Издатель публикует пост
    ↓
Все подписчики получают уведомление:
- Queue="user_feed_1"
- Queue="user_feed_2"
- Queue="notifications"
- Queue="analytics"
```

**Когда использовать:** Когда нужна рассылка одного сообщения множеству получателей (notifications, broadcasts).

---

### 3. **Topic Exchange** (Тематический)

**Логика:** Маршрутизация по паттерну Routing Key с поддержкой wildcards.

```
Wildcard patterns:
* = один элемент
# = любое количество элементов
. = разделитель
```

**Примеры:**

```
Bindings:
- Queue="errors",    routingKey="*.error"      (Все ошибки)
- Queue="logs",      routingKey="order.*"      (Все ордера)
- Queue="critical",  routingKey="#"            (Всё вообще)

Messages:
- routingKey="user.error"     → Queue="errors" ✓, Queue="critical" ✓
- routingKey="order.created"  → Queue="logs" ✓, Queue="critical" ✓
- routingKey="payment"        → Queue="critical" ✓
```

**Когда использовать:** Когда нужна гибкая фильтрация сообщений по паттернам (логирование, аналитика).

---

### 4. **Headers Exchange** (По заголовкам)

**Логика:** Маршрутизация по заголовкам сообщения (metadata), а не по Routing Key.

```
Сообщение имеет заголовки:
headers={
  "type": "premium",
  "region": "EU"
}

Exchange смотрит на заголовки и решает, в какую Queue его отправить
```

**Когда использовать:** Редко. Когда нужна сложная маршрутизация по множеству критериев.

---

## Ваш проект - практический пример

Вот визуализация того, как RabbitMQ используется в вашем проекте:

```
АРХИТЕКТУРА ПРОЕКТА
═══════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────┐
│ PRODUCER (приложение для отправки)                          │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ImageFileScanner                                           │
│  ├─ Сканирует папку: c:\image-demo\images                 │
│  └─ Находит изображение: photo.jpg                        │
│       ↓                                                     │
│  MessageSender                                              │
│  ├─ Создаёт ThumbnailCreationRequest объект               │
│  │  {                                                      │
│  │    "imageFilePath": "c:\\image-demo\\images\\photo.jpg",│
│  │    "thumbnailFilePath": "c:\\image-demo\\thumbnails\\..." │
│  │  }                                                      │
│  └─ Отправляет в RabbitMQ                                 │
│                                                             │
└──────────────────────────────────┬──────────────────────────┘
                                   │
                                   │ convertAndSend(
                                   │   "image.processing.exchange",
                                   │   "thumbnail.create",
                                   │   request
                                   │ )
                                   │
                                   ↓
┌─────────────────────────────────────────────────────────────┐
│ RABBITMQ SERVER (брокер)                                    │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  Exchange: "image.processing.exchange"                     │
│  Type: DirectExchange                                       │
│  ├─ Получает сообщение с routingKey="thumbnail.create"    │
│  ├─ Смотрит на Bindings                                    │
│  └─ Находит: routingKey="thumbnail.create" → Queue="..."  │
│                                               ↓           │
│  Queue: "image.processing.queue"                          │
│  ├─ Сообщение встаёт в очередь                           │
│  ├─ Ждёт, пока Consumer возьмёт его                       │
│  └─ [Сообщение 1] [Сообщение 2] [Сообщение 3] ← FIFO    │
│                                                             │
└──────────────────────────────────┬──────────────────────────┘
                                   │
                                   │ RabbitMQ отправляет
                                   │ сообщение Consumer
                                   │
                                   ↓
┌─────────────────────────────────────────────────────────────┐
│ CONSUMER (приложение для обработки)                         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│ MessageListener (@RabbitListener)                          │
│ ├─ Слушает очередь: "image.processing.queue"             │
│ ├─ Получает ThumbnailCreationRequest                      │
│ ├─ Использует Thumbnailator для создания thumbnail      │
│ │  (размер 200x200)                                        │
│ ├─ Сохраняет результат:                                  │
│ │  c:\image-demo\thumbnails\photo_thumb.jpg              │
│ └─ Отправляет ACK (подтверждение обработки)              │
│                                                             │
└─────────────────────────────────────────────────────────────┘

РЕЗУЛЬТАТ:
✓ Producer отправил сообщение (асинхронно, не ждал)
✓ RabbitMQ доставил сообщение
✓ Consumer обработал и создал thumbnail
✓ Оба приложения независимы друг от друга
```

---

## Архитектура сообщений

### Сообщение: ThumbnailCreationRequest

```java
// Это то, что передаётся между Producer и Consumer

public class ThumbnailCreationRequest {
    private String imageFilePath;
    private String thumbnailFilePath;
}
```

### Сериализация

Spring Boot автоматически конвертирует объект в JSON и обратно:

```
Java Object                JSON (в RabbitMQ)         Java Object
─────────────              ──────────────────        ───────────
{                          {                          {
  imageFilePath:  ──────→  "imageFilePath": ────→    imageFilePath:
  "photo.jpg"              "photo.jpg"               "photo.jpg"
}                          }                         }


MessageConverter (Jackson2JsonMessageConverter)
Это класс отвечает за эту конвертацию
```

**В конфигурации:**

```java
@Bean
public MessageConverter jsonMessageConverter() {
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.findAndRegisterModules();
    objectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
    return new Jackson2JsonMessageConverter(objectMapper);
}
```

---

## Коды примеров

### Соответствие между компонентами

```
КОМПОНЕНТ               МЕСТО В КОДЕ            НАЗНАЧЕНИЕ
──────────────────────────────────────────────────────────────────

Exchange                RabbitMqConfiguration   Создание DirectExchange
                        .directExchange()       "image.processing.exchange"

Queue                   RabbitMqConfiguration   Создание Queue
                        .queue()                "image.processing.queue"

Binding                 RabbitMqConfiguration   Связь между
                        .binding()              Exchange и Queue

Routing Key             application.yml         Правило маршрутизации
                        messagingProperties     "thumbnail.create"
                        .getRoutingKey()

Producer                MessageSender           Отправка сообщений
                        .sendMessage()          в Exchange

Consumer                MessageListener         Получение сообщений
                        .processImage()         из Queue
                        @RabbitListener

Message Converter       RabbitMqConfiguration   Конвертация
                        .jsonMessageConverter() Object ↔ JSON

RabbitTemplate          Внедряется Spring       Инструмент для
                        автоматически          отправки сообщений
```

### Producer: Как отправить сообщение

```java
// ШАГ 1: Создать сообщение
ThumbnailCreationRequest request = new ThumbnailCreationRequest();
request.setImageFilePath("c:\\images\\photo.jpg");
request.setThumbnailFilePath("c:\\thumbnails\\photo_thumb.jpg");

// ШАГ 2: Отправить в RabbitMQ
messageSender.sendMessage(request);

// ЧТО ПРОИЗОЙДЁТ:
// 1. request будет сериализован в JSON
// 2. Отправлен в Exchange "image.processing.exchange"
// 3. С Routing Key "thumbnail.create"
// 4. Exchange маршрутизирует в Queue "image.processing.queue"
// 5. Сообщение ждёт Consumer
```

### Consumer: Как получить сообщение

```java
@Component
public class MessageListener {
    
    // @RabbitListener - аннотация Spring для слушания очереди
    @RabbitListener(queues = "${messaging-config.queueName}")
    public void processImage(ThumbnailCreationRequest request) {
        // ЧТО ПРОИЗОЙДЁТ:
        // 1. RabbitMQ даст это сообщение Consumer
        // 2. Spring десериализует JSON в ThumbnailCreationRequest
        // 3. Вызовет этот метод с десериализованным объектом
        
        // ОБРАБОТКА:
        String imagePath = request.getImageFilePath();
        String thumbnailPath = request.getThumbnailFilePath();
        
        // Создание thumbnail
        Thumbnails.of(new File(imagePath))
                .size(200, 200)
                .toFile(new File(thumbnailPath));
        
        // Метод автоматически отправит ACK в RabbitMQ
        // Сообщение будет удалено из Queue
    }
}
```

### Конфигурация: Полная настройка

```java
@Configuration
public class RabbitMqConfiguration {
    
    // 1. Создаём Exchange
    @Bean
    public DirectExchange directExchange(MessagingProperties props) {
        // DirectExchange - получает Routing Key и маршрутизирует сообщение
        return new DirectExchange(props.getExchangeName());
        // Название: "image.processing.exchange"
    }
    
    // 2. Создаём Queue
    @Bean
    public Queue queue(MessagingProperties props) {
        // Queue - хранит сообщения
        return new Queue(props.getQueueName());
        // Название: "image.processing.queue"
    }
    
    // 3. Связываем Exchange и Queue
    @Bean
    public Binding binding(Queue queue, DirectExchange exchange, 
                          MessagingProperties props) {
        // Binding - указывает Exchange какие сообщения в какую Queue отправлять
        return BindingBuilder.bind(queue)
                .to(exchange)
                .with(props.getRoutingKey());
        // Правило: routingKey="thumbnail.create" → Queue
    }
    
    // 4. Конвертор JSON ↔ Object
    @Bean
    public MessageConverter jsonMessageConverter() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.findAndRegisterModules();
        // WRITE_DATES_AS_TIMESTAMPS = false означает:
        // Даты не будут преобразованы в timestamp, а в строку ISO
        mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
        return new Jackson2JsonMessageConverter(mapper);
    }
}
```

---

## Бестпрактики и советы

### 1. **Идемпотентность сообщений**

**Проблема:** RabbitMQ может отправить одно сообщение несколько раз (в случае сбоев).

**Решение:** Сделайте Consumer идемпотентным.

```java
// ❌ НЕПРАВИЛЬНО: Если сообщение придёт дважды, thumbnail создастся дважды
@RabbitListener(queues = "image.processing.queue")
public void processImage(ThumbnailCreationRequest request) {
    Thumbnails.of(request.getImageFilePath())
            .toFile(request.getThumbnailFilePath());
}

// ✅ ПРАВИЛЬНО: Проверяем, не создан ли уже
@RabbitListener(queues = "image.processing.queue")
public void processImage(ThumbnailCreationRequest request) {
    File thumbnail = new File(request.getThumbnailFilePath());
    if (!thumbnail.exists()) {
        Thumbnails.of(request.getImageFilePath())
                .toFile(thumbnail);
    }
}
```

---

### 2. **Обработка ошибок**

**Проблема:** Если Consumer выбросит исключение, что произойдёт с сообщением?

**Решение:** Используйте try-catch и логирование.

```java
@RabbitListener(queues = "image.processing.queue")
public void processImage(ThumbnailCreationRequest request) {
    try {
        // Обработка
        Thumbnails.of(request.getImageFilePath())
                .toFile(request.getThumbnailFilePath());
    } catch (Exception e) {
        // Логируем ошибку
        logger.error("Failed to process image: " + request, e);
        // RabbitMQ НЕ получит ACK и переотправит сообщение
        // Или выбросьте exception, если хотите retry
        throw new RuntimeException("Processing failed", e);
    }
}
```

---

### 3. **Мониторинг RabbitMQ**

RabbitMQ имеет встроенный веб-интерфейс для мониторинга:

```
URL: http://localhost:15672
Username: guest
Password: guest

Там вы можете:
- Видеть все Exchange и Queue
- Смотреть количество сообщений в очереди
- Видеть активных Consumer
- Проверить статус соединений
```

**В вашем docker-compose.yml:**

```yaml
ports:
  - 5672:5672      # AMQP порт (для реальных сообщений)
  - 15672:15672    # Веб-интерфейс (Management Plugin)
```

---

### 4. **Надёжность доставки**

RabbitMQ гарантирует доставку сообщений, используя ACK (подтверждение):

```
Producer отправляет
    ↓
RabbitMQ сохраняет в памяти / на диске
    ↓
Consumer получает сообщение
    ↓
Consumer обрабатывает
    ↓
Consumer отправляет ACK (подтверждение)
    ↓
RabbitMQ удаляет сообщение из Queue

Если Consumer упадёт ДО ACK:
→ RabbitMQ вернёт сообщение в очередь
→ Другой Consumer его обработает
```

---

### 5. **Типы сообщений**

Используйте разные Routing Key для разных типов задач:

```yaml
# application.yml
messaging-config:
  exchangeName: images.exchange
  routingKeys:
    - thumbnail.create
    - thumbnail.resize
    - thumbnail.delete
```

```java
// Отправка разных типов
messageSender.sendMessage(request, "thumbnail.create");
messageSender.sendMessage(request, "thumbnail.resize");
messageSender.sendMessage(request, "thumbnail.delete");

// Consumer может слушать несколько очередей
@RabbitListener(queues = "image.processing.queue")
public void processImage(Message message) {
    String routingKey = message.getMessageProperties().getReceivedRoutingKey();
    
    if ("thumbnail.create".equals(routingKey)) {
        // Создание
    } else if ("thumbnail.resize".equals(routingKey)) {
        // Изменение размера
    }
}
```

---

### 6. **Масштабирование Consumer**

Преимущество RabbitMQ - вы можете запустить несколько Consumer:

```
ОДИН Queue → НЕСКОЛЬКО Consumer

Queue: [Message1][Message2][Message3][Message4]
          ↓          ↓          ↓          ↓
      Consumer1  Consumer2  Consumer3  Consumer4

RabbitMQ АВТОМАТИЧЕСКИ распределит сообщения!
Каждый Consumer обработает одно сообщение в раз.
```

**Как это работает:**
- Запустите `consumer` приложение несколько раз
- Каждый экземпляр подключится к одной Queue
- RabbitMQ автоматически распределит нагрузку

---

### 7. **Persisting Messages (Сохранение на диск)**

По умолчанию RabbitMQ хранит сообщения в памяти. Если RabbitMQ упадёт, сообщения потеряются.

**Решение:** Используйте Durable Queue:

```java
@Bean
public Queue queue(MessagingProperties props) {
    return new Queue(props.getQueueName(), true);  // true = durable
    // теперь сообщения сохранятся на диск RabbitMQ
}
```

**Также сделайте сообщения persistent:**

```java
public void sendMessage(ThumbnailCreationRequest request) {
    rabbitTemplate.convertAndSend(
            messagingProperties.getExchangeName(),
            messagingProperties.getRoutingKey(),
            request,
            messagePostProcessor -> {
                messagePostProcessor.getMessageProperties()
                        .setDeliveryMode(MessageDeliveryMode.PERSISTENT);
                return messagePostProcessor;
            }
    );
}
```

---

### 8. **TTL (Time To Live) - Время жизни сообщения**

Можете задать, как долго сообщение может находиться в очереди:

```java
@Bean
public Queue queue(MessagingProperties props) {
    return QueueBuilder.durable(props.getQueueName())
            .ttl(300000)  // 5 минут = 300000 мс
            .build();
    // Если сообщение 5 минут в очереди и никто его не взял - удалится
}
```

---

### 9. **Dead Letter Exchange (DLX) - Очередь для "мёртвых" сообщений**

Если Consumer не может обработать сообщение, его можно отправить в DLX:

```java
@Bean
public Queue deadLetterQueue() {
    return new Queue("thumbnail.processing.dlq");
}

@Bean
public DirectExchange deadLetterExchange() {
    return new DirectExchange("thumbnail.processing.dlx");
}

@Bean
public Binding deadLetterBinding(Queue deadLetterQueue, 
                                  DirectExchange deadLetterExchange) {
    return BindingBuilder.bind(deadLetterQueue)
            .to(deadLetterExchange)
            .with("rejected");
}

@Bean
public Queue mainQueue() {
    return QueueBuilder.durable("image.processing.queue")
            .deadLetterExchange("thumbnail.processing.dlx")
            .deadLetterRoutingKey("rejected")
            .build();
}
```

---

### 10. **Мониторинг через логи**

Добавьте логирование в Consumer:

```java
private static final Logger logger = LoggerFactory.getLogger(MessageListener.class);

@RabbitListener(queues = "${messaging-config.queueName}")
public void processImage(ThumbnailCreationRequest request) {
    logger.info("Received message: {}", request);
    try {
        // Обработка
        logger.info("Successfully processed: {}", request.getImageFilePath());
    } catch (Exception e) {
        logger.error("Failed to process: {}", request, e);
    }
}
```

---

## Резюме: Основные концепции

| Концепция | Что это | Аналогия |
|-----------|--------|----------|
| **Producer** | Отправитель сообщений | Отправитель письма |
| **Consumer** | Получатель сообщений | Получатель письма |
| **Exchange** | Маршрутизатор сообщений | Почтовое отделение |
| **Queue** | Хранилище сообщений | Почтовый ящик |
| **Binding** | Правило маршрутизации | Адрес на письме |
| **Routing Key** | Ключ для маршрутизации | Индекс получателя |
| **Message** | Сообщение | Письмо |
| **ACK** | Подтверждение обработки | Квитанция о получении |

---

## Команды для работы с Docker

```bash
# Запустить RabbitMQ в Docker
docker-compose up -d

# Проверить статус
docker-compose ps

# Остановить RabbitMQ
docker-compose down

# Смотреть логи RabbitMQ
docker-compose logs -f rabbitmq

# Войти в контейнер
docker exec -it rabbitmq bash

# Проверить очереди через CLI
docker exec -it rabbitmq rabbitmqctl list_queues

# Проверить Exchange
docker exec -it rabbitmq rabbitmqctl list_exchanges
```

---

## Визуальная схема всего процесса

```
┌─────────────────────────────────────────────────────────────────┐
│                         ПОЛНАЯ АРХИТЕКТУРА                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  СЛОЙ PRODUCER (Отправитель)                                    │
│  ────────────────────────                                        │
│  ImageFileScanner → ThumbnailCreationRequest                     │
│                            ↓                                      │
│                   MessageSender.sendMessage()                    │
│                            ↓                                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ СЛОЙ RABBITMQ (Брокер)                                  │   │
│  ├──────────────────────────────────────────────────────────┤   │
│  │                                                            │   │
│  │  rabbitTemplate.convertAndSend(                           │   │
│  │      "image.processing.exchange",                         │   │
│  │      "thumbnail.create",                                  │   │
│  │      request                                              │   │
│  │  )                                                        │   │
│  │        ↓                                                  │   │
│  │  ┌───────────────────────────────────────────────┐       │   │
│  │  │ EXCHANGE (Direct)                             │       │   │
│  │  │ "image.processing.exchange"                   │       │   │
│  │  │                                               │       │   │
│  │  │ Получает: routingKey="thumbnail.create"      │       │   │
│  │  │ Смотрит: есть ли Binding для этого ключа?   │       │   │
│  │  └────────────┬────────────────────────────────┘       │   │
│  │               │                                         │   │
│  │               │ ДА! Binding связывает этот ключ      │   │
│  │               │ с Queue                               │   │
│  │               ↓                                        │   │
│  │  ┌───────────────────────────────────────────────┐       │   │
│  │  │ BINDING (Правило маршрутизации)               │       │   │
│  │  │ routingKey="thumbnail.create"                 │       │   │
│  │  │       ↓                                        │       │   │
│  │  │ Queue="image.processing.queue"                │       │   │
│  │  └────────────┬────────────────────────────────┘       │   │
│  │               │                                         │   │
│  │               │ Отправляем сюда                       │   │
│  │               ↓                                        │   │
│  │  ┌───────────────────────────────────────────────┐       │   │
│  │  │ QUEUE (Очередь)                               │       │   │
│  │  │ "image.processing.queue"                      │       │   │
│  │  │                                               │       │   │
│  │  │ [Message1 JSON][Message2 JSON] ← FIFO        │       │   │
│  │  │                                               │       │   │
│  │  │ Consumer слушает эту очередь...               │       │   │
│  │  └────────────┬────────────────────────────────┘       │   │
│  │               │                                         │   │
│  └───────────────┼─────────────────────────────────────────┘   │
│                  │                                               │
│                  │ RabbitMQ даёт сообщение Consumer           │
│                  ↓                                               │
│  СЛОЙ CONSUMER (Получатель)                                     │
│  ─────────────────────────                                      │
│  MessageListener                                                │
│      @RabbitListener(queues="image.processing.queue")           │
│      public void processImage(ThumbnailCreationRequest request)  │
│      {                                                           │
│          // Spring десериализует JSON → Object                 │
│          //                                                     │
│          // Обработка:                                         │
│          // 1. Читаем imagePath                                │
│          // 2. Создаём thumbnail через Thumbnailator         │
│          // 3. Сохраняем в thumbnailPath                      │
│          // 4. АВТОМАТИЧЕСКИ отправляем ACK                  │
│      }                                                          │
│                                                                   │
│  ACK (Подтверждение) отправляется обратно в RabbitMQ           │
│      ↓                                                            │
│  RabbitMQ: "ОК, сообщение обработано, удаляю из очереди"       │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘

РЕЗУЛЬТАТ: Быстрая, надёжная, асинхронная обработка сообщений! ✓
```

---

## Заключение

**RabbitMQ - это мощный инструмент для:**
- ✅ Развязывания приложений
- ✅ Асинхронной обработки задач
- ✅ Масштабирования систем
- ✅ Гарантирования доставки сообщений
- ✅ Разделения ответственности между компонентами

**В вашем проекте:**
- Producer сканирует папку с изображениями
- Отправляет сообщения в RabbitMQ
- Consumer получает сообщения и обрабатывает
- Оба приложения работают независимо

**Это классический паттерн обработки данных в масштабируемых системах!**

---

**Создано для обучения. Версия 1.0**

