# 📨 Полный гайд по RabbitMQ и Messaging для начинающих

---

## 📖 Содержание

1. [Что такое Messaging и зачем это нужно?](#1-что-такое-messaging-и-зачем-это-нужно)
2. [Основные понятия и термины](#2-основные-понятия-и-термины)
3. [Что такое RabbitMQ?](#3-что-такое-rabbitmq)
4. [Архитектура RabbitMQ](#4-архитектура-rabbitmq)
5. [Типы Exchange (обменников)](#5-типы-exchange-обменников)
6. [Паттерны обмена сообщениями](#6-паттерны-обмена-сообщениями)
7. [Установка RabbitMQ](#7-установка-rabbitmq)
8. [Spring Boot + RabbitMQ: Полная интеграция](#8-spring-boot--rabbitmq-полная-интеграция)
9. [Продвинутые темы](#9-продвинутые-темы)
10. [Типичные ошибки и их решения](#10-типичные-ошибки-и-их-решения)
11. [Сравнение с другими брокерами](#11-сравнение-с-другими-брокерами)
12. [Итоговая шпаргалка](#12-итоговая-шпаргалка)

---

## 1. Что такое Messaging и зачем это нужно?

### 🤔 Проблема: Синхронное взаимодействие

Представь, что ты заказываешь пиццу по телефону. Ты звонишь, ждёшь на линии, пока оператор не ответит, затем оформляете заказ, и только потом ты можешь заниматься своими делами. Это **синхронное** взаимодействие — ты **блокируешься** и ждёшь.

В программировании, когда **Сервис A** вызывает **Сервис B** напрямую (например, через HTTP REST):

```
Сервис A ---[HTTP запрос]---> Сервис B
Сервис A <--[HTTP ответ]---- Сервис B
         (А ждёт весь это время!)
```

**Проблемы такого подхода:**
- ❌ Если Сервис B упал — Сервис A тоже получит ошибку
- ❌ Если Сервис B медленный — Сервис A зависает и ждёт
- ❌ Если нагрузка выросла — оба сервиса страдают одновременно
- ❌ Сервисы **тесно связаны** (tight coupling) — изменение одного требует изменения другого

### ✅ Решение: Асинхронный обмен сообщениями

Теперь представь, что ты заказываешь пиццу через приложение. Ты отправляешь заказ и **сразу идёшь по своим делам**. Когда пицца будет готова — тебе придёт уведомление. Это **асинхронное** взаимодействие.

```
Сервис A ---[сообщение]---> [📬 Брокер сообщений] ---[сообщение]---> Сервис B
Сервис A продолжает работать!                         (когда готов)
```

**Преимущества:**
- ✅ **Loose coupling** — сервисы не знают друг о друге
- ✅ **Отказоустойчивость** — если Сервис B упал, сообщения накапливаются в очереди
- ✅ **Масштабируемость** — можно добавить несколько экземпляров Сервиса B
- ✅ **Пиковые нагрузки** — очередь выступает буфером

### 🌍 Реальные примеры использования

| Сценарий | Без Messaging | С Messaging |
|----------|--------------|-------------|
| Отправка email после регистрации | Пользователь ждёт пока письмо отправится | Задача кладётся в очередь, ответ мгновенный |
| Обработка платежей | Медленный банковский API блокирует UI | Запрос в очереди, уведомление придёт позже |
| Обновление нескольких сервисов | Нужно знать адреса всех сервисов | Публикуй событие — подписчики сами обработают |
| Конвертация видео | Пользователь ждёт минуты | Загрузил → очередь → уведомление по готовности |

---

## 2. Основные понятия и термины

### 📌 Ключевые термины

| Термин | Объяснение | Аналогия |
|--------|-----------|---------|
| **Message (Сообщение)** | Данные, которые передаются | Письмо |
| **Producer (Производитель)** | Тот, кто отправляет сообщения | Отправитель письма |
| **Consumer (Потребитель)** | Тот, кто получает и обрабатывает сообщения | Получатель письма |
| **Queue (Очередь)** | Буфер, где хранятся сообщения | Почтовый ящик |
| **Broker (Брокер)** | Посредник, который управляет очередями | Почтовое отделение |
| **Exchange (Обменник)** | Маршрутизатор сообщений (специфично для RabbitMQ) | Сортировочный центр |
| **Binding (Привязка)** | Правило: какой Exchange → какая Queue | Правило сортировки |
| **Routing Key** | Метка сообщения для маршрутизации | Адрес на конверте |
| **Acknowledgement (ACK)** | Подтверждение получения сообщения | Подпись о получении |

### 🔄 Жизненный цикл сообщения

```
1. Producer создаёт сообщение
        ↓
2. Отправляет в Exchange с routing key
        ↓
3. Exchange по правилам (bindings) кладёт в Queue
        ↓
4. Consumer читает сообщение из Queue
        ↓
5. Consumer отправляет ACK (подтверждение)
        ↓
6. RabbitMQ удаляет сообщение из Queue
```

---

## 3. Что такое RabbitMQ?

**RabbitMQ** — это **брокер сообщений** с открытым исходным кодом. Он реализует протокол **AMQP** (Advanced Message Queuing Protocol).

### 🐇 Почему именно RabbitMQ?

- **Надёжность** — сообщения не теряются даже при перезапуске
- **Гибкость маршрутизации** — мощная система Exchange/Binding
- **Простота** — удобный веб-интерфейс управления
- **Зрелость** — используется в продакшене с 2007 года
- **Интеграция** — отличная поддержка в Spring Framework
- **Протоколы** — поддерживает AMQP, MQTT, STOMP

### 🏢 Кто использует RabbitMQ?

Mozilla, Instagram, Reddit, Zalando и тысячи других компаний.

---

## 4. Архитектура RabbitMQ

```
┌─────────────────────────────────────────────────────────┐
│                    RabbitMQ Broker                       │
│                                                          │
│  ┌──────────┐    ┌──────────────┐    ┌───────────────┐  │
│  │ Virtual  │    │   Exchange   │    │    Queue      │  │
│  │  Host    │───>│  (маршрутиз.)│───>│  (очередь)    │  │
│  │ (vhost)  │    │              │    │               │  │
│  └──────────┘    └──────────────┘    └───────────────┘  │
│                        │                    │            │
│                    Binding              Binding          │
│                   (правила)            (правила)         │
└─────────────────────────────────────────────────────────┘
        ↑                                       ↓
   Producer                               Consumer
  (отправляет)                           (получает)
```

### 🏠 Virtual Host (vhost)

Виртуальный хост — это логическое разделение внутри одного RabbitMQ сервера. Как разные папки на диске. Используется для изоляции разных приложений или сред (dev/staging/prod).

По умолчанию используется vhost `/`.

---

## 5. Типы Exchange (обменников)

Exchange принимает сообщение от Producer и решает, в какую очередь (или очереди) его положить.

### 1️⃣ Direct Exchange (Прямой)

**Принцип:** Сообщение идёт в очередь, чей binding key **точно совпадает** с routing key сообщения.

```
Producer --[routing_key="email"]--> Direct Exchange
                                          |
                          ┌───────────────┴──────────────┐
                          ↓                               ↓
                  [binding: "email"]              [binding: "sms"]
                   Queue: email_queue              Queue: sms_queue
                  ✅ Совпадение!                   ❌ Не совпадает
```

**Когда использовать:** Точная маршрутизация на конкретную очередь.

### 2️⃣ Fanout Exchange (Широковещательный)

**Принцип:** Сообщение рассылается **во все** привязанные очереди. Routing key **игнорируется**.

```
Producer --[любой ключ]--> Fanout Exchange
                                  |
                   ┌──────────────┼──────────────┐
                   ↓              ↓              ↓
              Queue_1          Queue_2         Queue_3
           ✅ Получит       ✅ Получит      ✅ Получит
```

**Когда использовать:** Уведомления, которые должны получить все (например, "товар в наличии" — и email-сервис, и push-сервис, и SMS-сервис).

### 3️⃣ Topic Exchange (Тематический)

**Принцип:** Routing key — это строка с точками (`order.created.europe`). Поддерживает **шаблоны**:
- `*` — заменяет **одно** слово
- `#` — заменяет **ноль или более** слов

```
Producer --[routing_key="order.created.europe"]--> Topic Exchange
                                                          |
                              ┌───────────────────────────┤
                              ↓                           ↓
                   binding: "order.*.europe"     binding: "order.#"
                   ✅ Совпадение!               ✅ Совпадение!
                   
                              ↓
                   binding: "order.*.asia"
                   ❌ Не совпадает
```

**Когда использовать:** Гибкая маршрутизация по категориям событий.

### 4️⃣ Headers Exchange (По заголовкам)

**Принцип:** Маршрутизация по **заголовкам** сообщения (ключ-значение), а не по routing key.

**Когда использовать:** Сложная маршрутизация по множеству критериев (редко используется).

### 📊 Сводная таблица

| Тип | Routing Key | Шаблоны | Использование |
|-----|------------|---------|---------------|
| Direct | Точное совпадение | Нет | Простая маршрутизация |
| Fanout | Игнорируется | Нет | Broadcast всем |
| Topic | Шаблон с `*` и `#` | Да | Гибкая маршрутизация |
| Headers | По заголовкам | Нет | Сложные условия |

---

## 6. Паттерны обмена сообщениями

### 📤 1. Work Queue (Очередь задач)

Один producer, несколько consumers. Каждое сообщение обрабатывается **только одним** consumer.

```
Producer → [Queue] → Consumer 1
                   → Consumer 2
                   → Consumer 3
```

**Случай использования:** Балансировка нагрузки при обработке задач (конвертация файлов, отправка писем).

### 📡 2. Publish/Subscribe (Публикация/Подписка)

Один producer публикует событие — **все** подписчики его получают.

```
Producer → [Fanout Exchange] → Queue 1 → Consumer 1
                             → Queue 2 → Consumer 2
                             → Queue 3 → Consumer 3
```

**Случай использования:** Событие "новый пользователь зарегистрировался" → email-сервис, аналитика, CRM.

### 🔄 3. Request/Reply (Запрос/Ответ)

Producer отправляет запрос и ждёт ответ (асинхронный RPC).

```
Client → [request_queue] → Server
Client ← [reply_queue]   ← Server
```

### 🪣 4. Dead Letter Queue (Очередь мёртвых писем)

Сообщения, которые не удалось обработать, перекладываются в специальную очередь для анализа.

```
[Main Queue] → Consumer (ошибка или TTL истёк) → [Dead Letter Exchange] → [DLQ]
```

---

## 7. Установка RabbitMQ

### 🐳 Способ 1: Docker (Рекомендуется!)

Это самый простой способ. Убедись, что Docker установлен.

```bash
# Запуск RabbitMQ с веб-интерфейсом управления
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3-management
```

После запуска:
- **AMQP порт:** `localhost:5672` (для приложений)
- **Веб-интерфейс:** `http://localhost:15672`
- **Логин/пароль по умолчанию:** `guest` / `guest`

### 💻 Способ 2: Установка на Windows

1. Установи **Erlang** (язык, на котором написан RabbitMQ): https://www.erlang.org/downloads
2. Скачай RabbitMQ installer: https://www.rabbitmq.com/install-windows.html
3. Запусти установщик
4. Включи веб-плагин: `rabbitmq-plugins enable rabbitmq_management`

### 🌐 Веб-интерфейс Management UI

После запуска открой `http://localhost:15672`:

```
┌─────────────────────────────────┐
│     RabbitMQ Management         │
│                                 │
│  Overview | Connections | ...   │
│  ─────────────────────────────  │
│  Queues: 3  Messages: 125       │
│  Consumers: 5  Rate: 12 msg/s   │
└─────────────────────────────────┘
```

Здесь можно:
- 👀 Просматривать очереди и сообщения
- 📊 Смотреть статистику
- 🔧 Создавать/удалять очереди и exchanges
- 📨 Вручную публиковать сообщения для тестирования

---

## 8. Spring Boot + RabbitMQ: Полная интеграция

### 📦 Шаг 1: Создание проекта и зависимости

Создай Spring Boot проект. В `pom.xml` добавь зависимость:

```xml
<dependencies>
    <!-- Spring Boot Starter для RabbitMQ -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    
    <!-- Для веб-части (если нужен REST контроллер) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Lombok для удобства (опционально) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    
    <!-- Для Jackson (сериализация объектов в JSON) -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
</dependencies>
```

### ⚙️ Шаг 2: Настройка application.properties

```properties
# Подключение к RabbitMQ
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.virtual-host=/

# Настройки для listener (потребителя)
spring.rabbitmq.listener.simple.acknowledge-mode=manual
spring.rabbitmq.listener.simple.prefetch=10

# Для удобства логирования
logging.level.org.springframework.amqp=DEBUG
```

Или `application.yml`:

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual
        prefetch: 10
```

### 🔧 Шаг 3: Конфигурация (RabbitMQ Config)

Создаём конфигурационный класс, который объявляет Exchange, Queue и Binding:

```java
package com.example.rabbitmqdemo.config;

import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMQConfig {

    // ============================================================
    // 1. Константы — имена очередей, обменников, ключей маршрутизации
    // ============================================================
    
    public static final String QUEUE_NAME = "order.queue";
    public static final String EXCHANGE_NAME = "order.exchange";
    public static final String ROUTING_KEY = "order.created";
    
    // Dead Letter Queue (для сообщений с ошибками)
    public static final String DLQ_NAME = "order.dlq";
    public static final String DLQ_EXCHANGE = "order.dlx";
    public static final String DLQ_ROUTING_KEY = "order.dead";

    // ============================================================
    // 2. Объявление Queue (очереди)
    // ============================================================
    
    @Bean
    public Queue orderQueue() {
        return QueueBuilder.durable(QUEUE_NAME)        // durable = сохраняется при перезапуске
            .withArgument("x-dead-letter-exchange", DLQ_EXCHANGE)      // DLX настройка
            .withArgument("x-dead-letter-routing-key", DLQ_ROUTING_KEY) // DLX ключ
            .withArgument("x-message-ttl", 60000)     // TTL: сообщение живёт 60 секунд
            .build();
    }
    
    // Dead Letter Queue
    @Bean
    public Queue deadLetterQueue() {
        return QueueBuilder.durable(DLQ_NAME).build();
    }

    // ============================================================
    // 3. Объявление Exchange (обменника)
    // ============================================================
    
    @Bean
    public DirectExchange orderExchange() {
        return ExchangeBuilder.directExchange(EXCHANGE_NAME)
            .durable(true)  // сохраняется при перезапуске
            .build();
    }
    
    // Dead Letter Exchange
    @Bean
    public DirectExchange deadLetterExchange() {
        return ExchangeBuilder.directExchange(DLQ_EXCHANGE).durable(true).build();
    }

    // ============================================================
    // 4. Binding — связываем Queue с Exchange
    // ============================================================
    
    @Bean
    public Binding orderBinding(Queue orderQueue, DirectExchange orderExchange) {
        return BindingBuilder
            .bind(orderQueue)          // Очередь
            .to(orderExchange)         // к Exchange
            .with(ROUTING_KEY);        // по ключу маршрутизации
    }
    
    @Bean
    public Binding dlqBinding(Queue deadLetterQueue, DirectExchange deadLetterExchange) {
        return BindingBuilder.bind(deadLetterQueue).to(deadLetterExchange).with(DLQ_ROUTING_KEY);
    }

    // ============================================================
    // 5. MessageConverter — конвертируем Java объекты в JSON
    // ============================================================
    
    @Bean
    public MessageConverter jsonMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }

    // ============================================================
    // 6. RabbitTemplate — основной инструмент для отправки сообщений
    // ============================================================
    
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(jsonMessageConverter()); // используем JSON
        return template;
    }
}
```

### 📤 Шаг 4: Producer (Отправитель сообщений)

```java
package com.example.rabbitmqdemo.producer;

import com.example.rabbitmqdemo.config.RabbitMQConfig;
import com.example.rabbitmqdemo.model.OrderEvent;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Service;

@Slf4j
@Service
@RequiredArgsConstructor
public class OrderProducer {

    private final RabbitTemplate rabbitTemplate;

    /**
     * Отправляет событие о создании заказа в RabbitMQ.
     * 
     * @param orderEvent данные заказа
     */
    public void sendOrderCreatedEvent(OrderEvent orderEvent) {
        log.info("📤 Отправляем сообщение: orderId={}", orderEvent.getOrderId());
        
        rabbitTemplate.convertAndSend(
            RabbitMQConfig.EXCHANGE_NAME,  // В какой Exchange
            RabbitMQConfig.ROUTING_KEY,    // С каким ключом маршрутизации
            orderEvent                     // Само сообщение (сериализуется в JSON)
        );
        
        log.info("✅ Сообщение успешно отправлено");
    }
    
    /**
     * Отправка с дополнительными заголовками
     */
    public void sendWithHeaders(OrderEvent orderEvent) {
        rabbitTemplate.convertAndSend(
            RabbitMQConfig.EXCHANGE_NAME,
            RabbitMQConfig.ROUTING_KEY,
            orderEvent,
            message -> {
                // Добавляем кастомные заголовки
                message.getMessageProperties().setHeader("source", "web-app");
                message.getMessageProperties().setHeader("version", "1.0");
                message.getMessageProperties().setPriority(5);
                return message;
            }
        );
    }
}
```

### 📦 Модель сообщения

```java
package com.example.rabbitmqdemo.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;
import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class OrderEvent {
    private String orderId;
    private String customerId;
    private String status;
    private Double totalAmount;
    private List<String> items;
    private LocalDateTime createdAt;
}
```

### 📥 Шаг 5: Consumer (Получатель сообщений)

```java
package com.example.rabbitmqdemo.consumer;

import com.example.rabbitmqdemo.config.RabbitMQConfig;
import com.example.rabbitmqdemo.model.OrderEvent;
import com.rabbitmq.client.Channel;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Service;

import java.io.IOException;

@Slf4j
@Service
public class OrderConsumer {

    /**
     * Простейший listener — автоматическое подтверждение.
     * Spring сам десериализует JSON обратно в OrderEvent.
     */
    @RabbitListener(queues = RabbitMQConfig.QUEUE_NAME)
    public void handleOrderEvent(OrderEvent orderEvent) {
        log.info("📥 Получено сообщение: orderId={}, status={}",
            orderEvent.getOrderId(), orderEvent.getStatus());
        
        // Твоя бизнес-логика здесь
        processOrder(orderEvent);
        
        // ACK отправляется автоматически при успешном выходе из метода
    }
    
    /**
     * Listener с ручным подтверждением (Manual ACK).
     * Используй, когда нужна полная надёжность.
     */
    @RabbitListener(queues = RabbitMQConfig.QUEUE_NAME, ackMode = "MANUAL")
    public void handleWithManualAck(
            OrderEvent orderEvent,
            Message message,
            Channel channel) throws IOException {
        
        long deliveryTag = message.getMessageProperties().getDeliveryTag();
        
        try {
            log.info("📥 Обрабатываем orderId={}", orderEvent.getOrderId());
            
            processOrder(orderEvent);
            
            // ✅ Подтверждаем успешную обработку
            // false = подтверждаем только это одно сообщение
            channel.basicAck(deliveryTag, false);
            log.info("✅ ACK отправлен для deliveryTag={}", deliveryTag);
            
        } catch (Exception e) {
            log.error("❌ Ошибка при обработке orderId={}: {}", 
                orderEvent.getOrderId(), e.getMessage());
            
            // ❌ Отклоняем сообщение
            // deliveryTag - тег сообщения
            // false - не группировать
            // true - вернуть обратно в очередь (requeue)
            // false вместо true - отправить в Dead Letter Queue
            channel.basicNack(deliveryTag, false, false); // → в DLQ
        }
    }
    
    /**
     * Batch listener — обрабатывает несколько сообщений за раз.
     */
    @RabbitListener(queues = RabbitMQConfig.QUEUE_NAME, containerFactory = "batchListenerFactory")
    public void handleBatch(List<OrderEvent> orders) {
        log.info("📦 Получен батч из {} заказов", orders.size());
        orders.forEach(this::processOrder);
    }

    private void processOrder(OrderEvent orderEvent) {
        // Имитируем обработку
        log.info("⚙️ Обрабатываем заказ: {}", orderEvent.getOrderId());
        // ... бизнес-логика
    }
}
```

### 🌐 Шаг 6: REST Controller для тестирования

```java
package com.example.rabbitmqdemo.controller;

import com.example.rabbitmqdemo.model.OrderEvent;
import com.example.rabbitmqdemo.producer.OrderProducer;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.time.LocalDateTime;
import java.util.Arrays;
import java.util.UUID;

@RestController
@RequestMapping("/api/orders")
@RequiredArgsConstructor
public class OrderController {

    private final OrderProducer orderProducer;

    @PostMapping
    public ResponseEntity<String> createOrder(@RequestBody OrderEvent order) {
        // Генерируем ID если не передан
        if (order.getOrderId() == null) {
            order.setOrderId(UUID.randomUUID().toString());
        }
        order.setCreatedAt(LocalDateTime.now());
        order.setStatus("CREATED");
        
        // Отправляем в RabbitMQ
        orderProducer.sendOrderCreatedEvent(order);
        
        return ResponseEntity.ok("Заказ " + order.getOrderId() + " принят в обработку!");
    }
    
    @GetMapping("/test")
    public ResponseEntity<String> sendTestMessage() {
        OrderEvent testOrder = new OrderEvent(
            UUID.randomUUID().toString(),
            "customer-123",
            "CREATED",
            1500.0,
            Arrays.asList("item1", "item2"),
            LocalDateTime.now()
        );
        
        orderProducer.sendOrderCreatedEvent(testOrder);
        return ResponseEntity.ok("Тестовое сообщение отправлено!");
    }
}
```

### 🏁 Шаг 7: Главный класс приложения

```java
package com.example.rabbitmqdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RabbitMqDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(RabbitMqDemoApplication.class, args);
    }
}
```

### 📁 Структура проекта

```
src/main/java/com/example/rabbitmqdemo/
├── RabbitMqDemoApplication.java     ← Точка входа
├── config/
│   └── RabbitMQConfig.java          ← Конфигурация Exchange/Queue/Binding
├── model/
│   └── OrderEvent.java              ← Модель сообщения
├── producer/
│   └── OrderProducer.java           ← Отправка сообщений
├── consumer/
│   └── OrderConsumer.java           ← Получение сообщений
└── controller/
    └── OrderController.java         ← REST API для тестирования
```

---

## 9. Продвинутые темы

### 🔁 Retry (Повторная отправка при ошибке)

Настройка автоматических повторов при ошибке обработки:

```java
// В RabbitMQConfig.java добавляем:

@Bean
public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(
        ConnectionFactory connectionFactory,
        MessageConverter jsonMessageConverter) {
    
    SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory);
    factory.setMessageConverter(jsonMessageConverter);
    
    // Настройки retry
    factory.setAdviceChain(
        RetryInterceptorBuilder.stateless()
            .maxAttempts(3)              // максимум 3 попытки
            .backOffOptions(
                1000,   // начальная задержка 1 сек
                2.0,    // множитель (exponential backoff)
                10000   // максимальная задержка 10 сек
            )
            .recoverer(new RejectAndDontRequeueRecoverer()) // после всех попыток → в DLQ
            .build()
    );
    
    return factory;
}
```

### 💾 Персистентность (Сохранение сообщений)

Чтобы сообщения **не потерялись** при перезапуске RabbitMQ:

1. **Queue должна быть `durable`** (уже есть в нашем конфиге)
2. **Exchange должен быть `durable`** (уже есть)
3. **Сообщения должны быть `persistent`:**

```java
rabbitTemplate.convertAndSend(exchange, routingKey, message, msg -> {
    msg.getMessageProperties().setDeliveryMode(MessageDeliveryMode.PERSISTENT);
    return msg;
});
```

### 📊 Prefetch (Предварительная выборка)

`prefetch` — сколько сообщений Consumer берёт заранее, не дожидаясь ACK:

```properties
# Каждый consumer берёт не более 5 сообщений одновременно
spring.rabbitmq.listener.simple.prefetch=5
```

**Совет:** При тяжёлой обработке ставь `prefetch=1`, чтобы нагрузка равномерно распределялась между consumers.

### 🎯 Fanout Exchange: Публикация событий

```java
// Config
@Bean
public FanoutExchange notificationExchange() {
    return new FanoutExchange("notification.fanout");
}

@Bean
public Queue emailQueue() { return new Queue("notification.email"); }

@Bean
public Queue smsQueue() { return new Queue("notification.sms"); }

@Bean
public Queue pushQueue() { return new Queue("notification.push"); }

@Bean
public Binding emailBinding() {
    return BindingBuilder.bind(emailQueue()).to(notificationExchange());
}
@Bean
public Binding smsBinding() {
    return BindingBuilder.bind(smsQueue()).to(notificationExchange());
}
@Bean
public Binding pushBinding() {
    return BindingBuilder.bind(pushQueue()).to(notificationExchange());
}

// Producer — отправить всем
rabbitTemplate.convertAndSend("notification.fanout", "", event);
//                                                    ↑
//                               routing key не нужен для Fanout
```

### 🌿 Topic Exchange: Гибкая маршрутизация

```java
// Config
@Bean
public TopicExchange orderTopicExchange() {
    return new TopicExchange("order.topic");
}

// Binding: Европейские заказы
@Bean
public Binding europeBinding() {
    return BindingBuilder.bind(europeQueue())
        .to(orderTopicExchange())
        .with("order.*.europe");  // order.created.europe, order.updated.europe...
}

// Binding: Все срочные заказы
@Bean
public Binding urgentBinding() {
    return BindingBuilder.bind(urgentQueue())
        .to(orderTopicExchange())
        .with("order.urgent.#"); // order.urgent.europe, order.urgent.asia...
}

// Producer
rabbitTemplate.convertAndSend("order.topic", "order.created.europe", event);
// → попадёт в оба: europeQueue И urgentQueue (если ключ совпал с обоими)
```

### 🔒 Транзакции и надёжность

```java
// Publisher Confirms — RabbitMQ подтверждает получение сообщения
@Bean
public RabbitTemplate reliableRabbitTemplate(ConnectionFactory connectionFactory) {
    RabbitTemplate template = new RabbitTemplate(connectionFactory);
    template.setMessageConverter(jsonMessageConverter());
    
    // Включаем подтверждения от сервера
    template.setConfirmCallback((correlationData, ack, cause) -> {
        if (ack) {
            log.info("✅ RabbitMQ подтвердил получение сообщения");
        } else {
            log.error("❌ RabbitMQ НЕ получил сообщение: {}", cause);
            // Здесь можно повторить отправку или сохранить в БД
        }
    });
    
    // Returns — если сообщение не попало ни в одну очередь
    template.setReturnsCallback(returned -> {
        log.error("📭 Сообщение не доставлено ни в одну очередь: {}", 
            returned.getRoutingKey());
    });
    
    template.setMandatory(true); // Необходимо для returns
    return template;
}
```

---

## 10. Типичные ошибки и их решения

### ❌ Ошибка 1: Connection refused

```
com.rabbitmq.client.AlreadyClosedException: connection is already closed
org.springframework.amqp.AmqpConnectException: java.net.ConnectException: Connection refused
```

**Причина:** RabbitMQ не запущен или неверный хост/порт.

**Решение:**
```bash
# Проверь, запущен ли RabbitMQ
docker ps | grep rabbitmq

# Или запусти:
docker start rabbitmq
```

### ❌ Ошибка 2: Сообщения теряются при перезапуске

**Причина:** Queue или Exchange объявлены без `durable(true)`.

**Решение:** Всегда используй `QueueBuilder.durable()` и `ExchangeBuilder.durable(true)`.

### ❌ Ошибка 3: Infinite loop (бесконечный цикл обработки ошибок)

**Причина:** Consumer кидает исключение → сообщение возвращается в очередь → Consumer снова его берёт → снова ошибка.

**Решение:** Используй Dead Letter Queue и `channel.basicNack(deliveryTag, false, false)` (requeue=false).

### ❌ Ошибка 4: ClassCastException при десериализации

```
ClassCastException: LinkedHashMap cannot be cast to OrderEvent
```

**Причина:** Jackson не знает, в какой тип десериализовать.

**Решение:**
```java
// Настрой Jackson2JsonMessageConverter с типовой информацией
@Bean
public MessageConverter jsonMessageConverter() {
    Jackson2JsonMessageConverter converter = new Jackson2JsonMessageConverter();
    DefaultJackson2JavaTypeMapper typeMapper = new DefaultJackson2JavaTypeMapper();
    typeMapper.setTrustedPackages("com.example.*");
    converter.setJavaTypeMapper(typeMapper);
    return converter;
}
```

### ❌ Ошибка 5: Очередь не создаётся

**Причина:** Конфигурационные бины не регистрируются или опечатка в имени.

**Решение:** Убедись, что конфигурационный класс помечен `@Configuration` и бины помечены `@Bean`. Проверь через Management UI.

### ⚠️ Частые антипаттерны

| Антипаттерн | Проблема | Правильный подход |
|-------------|---------|------------------|
| Один Queue для всего | Смешивание разных типов сообщений | Отдельная очередь для каждого типа событий |
| Нет DLQ | Потеря сообщений с ошибками | Всегда настраивай Dead Letter Queue |
| Нет ACK | Потеря сообщений при краше consumer | Всегда подтверждай обработку |
| Большие сообщения | Нагрузка на сеть и память | Храни данные в БД, передавай только ID |
| Не логировать | Сложность отладки | Логируй messageId, orderId при получении/отправке |

---

## 11. Сравнение с другими брокерами

| Характеристика | RabbitMQ | Apache Kafka | ActiveMQ |
|---------------|---------|------------|---------|
| **Протокол** | AMQP | Свой | AMQP, OpenWire |
| **Маршрутизация** | Гибкая (Exchange/Binding) | По топикам | Очереди/Топики |
| **Сохранение** | Временно (до ACK) | Долго (лог) | Временно |
| **Скорость** | Очень высокая | Экстремально высокая | Высокая |
| **Порядок** | FIFO в очереди | Гарантирован в партиции | FIFO |
| **Сложность** | Средняя | Высокая | Низкая |
| **Идеально для** | Task queues, RPC, routing | Event streaming, Big Data | Простые задачи |

### Когда выбирать RabbitMQ:
- ✅ Нужна гибкая маршрутизация
- ✅ Task queues (распределение задач)
- ✅ Быстрое подтверждение доставки
- ✅ Небольшие/средние нагрузки

### Когда выбирать Kafka:
- ✅ Огромные объёмы данных (миллионы сообщений/сек)
- ✅ Event sourcing (история событий важна)
- ✅ Аналитика в реальном времени

---

## 12. Итоговая шпаргалка

### 🔑 Ключевые аннотации и классы

```java
// ===== КОНФИГУРАЦИЯ =====
@Configuration              // Класс конфигурации
@Bean Queue/Exchange/Binding // Объявление компонентов

// ===== PRODUCER =====
RabbitTemplate              // Основной класс для отправки
.convertAndSend(exchange, routingKey, object)  // Отправить объект

// ===== CONSUMER =====
@RabbitListener(queues = "my.queue")  // Подписка на очередь
@RabbitListener(queues = "q", ackMode = "MANUAL")  // Ручной ACK

channel.basicAck(tag, false)   // Подтвердить успех
channel.basicNack(tag, false, false)  // Отклонить → DLQ
channel.basicNack(tag, false, true)   // Отклонить → вернуть в очередь

// ===== EXCHANGE BUILDER =====
QueueBuilder.durable("name").build()
ExchangeBuilder.directExchange("name").durable(true).build()
BindingBuilder.bind(queue).to(exchange).with(routingKey)
```

### 🎯 Чеклист для продакшена

- [ ] Queue и Exchange объявлены как `durable`
- [ ] Сообщения отправляются как `PERSISTENT`
- [ ] Настроена Dead Letter Queue (DLQ)
- [ ] Consumer использует Manual ACK для важных операций
- [ ] Настроен retry с exponential backoff
- [ ] Prefetch настроен под нагрузку
- [ ] Логирование messageId/correlationId
- [ ] Мониторинг через Management UI или Prometheus
- [ ] Пароль `guest` изменён на боевой

### 📚 Полезные ресурсы

- 🌐 [Официальная документация RabbitMQ](https://www.rabbitmq.com/documentation.html)
- 🌐 [Spring AMQP Reference](https://docs.spring.io/spring-amqp/docs/current/reference/html/)
- 🎓 [RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html) — официальные туториалы на разных языках
- 🐳 [RabbitMQ Docker Hub](https://hub.docker.com/_/rabbitmq)

---

## 🚀 Быстрый старт: Минимальный рабочий пример

Если хочешь быстро попробовать, вот минимальный код:

**1. Запусти RabbitMQ:**
```bash
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

**2. pom.xml:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

**3. application.properties:**
```properties
spring.rabbitmq.host=localhost
```

**4. Весь код в одном файле:**
```java
@SpringBootApplication
public class App implements CommandLineRunner {

    @Bean
    Queue myQueue() { return new Queue("hello"); }

    @Autowired
    RabbitTemplate rabbit;

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

    @Override
    public void run(String... args) {
        // Отправляем
        rabbit.convertAndSend("hello", "Привет, RabbitMQ! 🐇");
        System.out.println("Сообщение отправлено!");
    }

    // Получаем
    @RabbitListener(queues = "hello")
    public void listen(String message) {
        System.out.println("Получено: " + message);
    }
}
```

Запускаешь — видишь в консоли:
```
Сообщение отправлено!
Получено: Привет, RabbitMQ! 🐇
```

**Поздравляю — ты только что отправил своё первое сообщение через RabbitMQ! 🎉**

