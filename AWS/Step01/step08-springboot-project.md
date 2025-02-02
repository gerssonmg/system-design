
O que acha? Queremos começar integrando o **SQS ao Spring Boot**? 🚀

> Sim

Ótimo! Agora vamos configurar o **Spring Boot** para consumir mensagens do **SQS** e publicar no **SNS**.  

---

## **📌 Passo 1: Criar o Projeto Spring Boot**
Crie um novo projeto com Spring Boot usando **Spring Initializr**:  
🔗 [https://start.spring.io](https://start.spring.io/)  

### **Selecione as Dependências:**
✅ **Spring Web** → Para criar APIs  
✅ **Spring Boot DevTools** → Para desenvolvimento rápido  
✅ **Spring Data JPA** → Para interação com o PostgreSQL  
✅ **PostgreSQL Driver** → Para conectar ao banco  
✅ **Spring Cloud AWS Messaging** → Para integrar com **SQS e SNS**  

📌 **Gere o projeto e extraia os arquivos**.

---

## **📌 Passo 2: Configurar `application.yml`**
Abra `src/main/resources/application.yml` e configure:

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://notification-db.<AWS_REGION>.rds.amazonaws.com:5432/postgres
    username: admin
    password: MinhaSenha123
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

cloud:
  aws:
    credentials:
      access-key: YOUR_ACCESS_KEY
      secret-key: YOUR_SECRET_KEY
    region:
      static: us-east-1
    stack:
      auto: false
    endpoint:
      sqs: https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT_ID/notification-queue
      sns: https://sns.us-east-1.amazonaws.com/YOUR_ACCOUNT_ID/notification-topic
```

---

## **📌 Passo 3: Criar o Modelo de Notificação**
Crie `src/main/java/com/example/notification/model/Notification.java`:

```java
package com.example.notification.model;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "notifications")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Notification {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String message;
    private String recipient;
}
```

---

## **📌 Passo 4: Criar o Repositório**
Crie `src/main/java/com/example/notification/repository/NotificationRepository.java`:

```java
package com.example.notification.repository;

import com.example.notification.model.Notification;
import org.springframework.data.jpa.repository.JpaRepository;

public interface NotificationRepository extends JpaRepository<Notification, Long> {
}
```

---

## **📌 Passo 5: Criar o Consumer para SQS**
Crie `src/main/java/com/example/notification/service/SqsConsumerService.java`:

```java
package com.example.notification.service;

import com.amazonaws.services.sqs.AmazonSQSAsync;
import com.amazonaws.services.sqs.model.Message;
import com.example.notification.model.Notification;
import com.example.notification.repository.NotificationRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.aws.messaging.listener.annotation.SqsListener;
import org.springframework.stereotype.Service;

@Service
@Slf4j
@RequiredArgsConstructor
public class SqsConsumerService {

    private final NotificationRepository repository;
    private final AmazonSQSAsync amazonSQSAsync;

    @SqsListener("notification-queue")
    public void receiveMessage(Message message) {
        log.info("📩 Mensagem recebida: {}", message.getBody());

        Notification notification = new Notification();
        notification.setMessage(message.getBody());
        notification.setRecipient("user@example.com");

        repository.save(notification);

        // Após processar, remover mensagem da fila
        amazonSQSAsync.deleteMessage("https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT_ID/notification-queue", message.getReceiptHandle());
    }
}
```

---

## **📌 Passo 6: Criar o Publisher para SNS**
Crie `src/main/java/com/example/notification/service/SnsPublisherService.java`:

```java
package com.example.notification.service;

import com.amazonaws.services.sns.AmazonSNS;
import com.amazonaws.services.sns.model.PublishRequest;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Service
@Slf4j
@RequiredArgsConstructor
public class SnsPublisherService {

    private final AmazonSNS amazonSNS;

    public void sendNotification(String message) {
        PublishRequest request = new PublishRequest()
                .withTopicArn("arn:aws:sns:us-east-1:YOUR_ACCOUNT_ID:notification-topic")
                .withMessage(message);

        amazonSNS.publish(request);
        log.info("📢 Notificação enviada para SNS: {}", message);
    }
}
```

---

## **📌 Passo 7: Criar API REST para Testes**
Crie `src/main/java/com/example/notification/controller/NotificationController.java`:

```java
package com.example.notification.controller;

import com.example.notification.service.SnsPublisherService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/notifications")
@RequiredArgsConstructor
public class NotificationController {

    private final SnsPublisherService snsPublisherService;

    @PostMapping
    public String sendNotification(@RequestParam String message) {
        snsPublisherService.sendNotification(message);
        return "✅ Notificação enviada!";
    }
}
```

---

## **📌 Passo 8: Rodar a Aplicação**
No terminal, dentro do projeto, rode:

```bash
mvn spring-boot:run
```

Agora, teste a API enviando uma notificação:

```bash
curl -X POST "http://localhost:8080/notifications?message=HelloAWS"
```

Se tudo estiver certo:  
✅ A mensagem será **enviada para o SNS**.  
✅ SNS **publicará no SQS**.  
✅ O **Consumer do SQS** processará e salvará no PostgreSQL.

---

## **🚀 Conclusão**
Agora temos um **serviço de notificações completo**!  

✅ Spring Boot integrado com **SQS + SNS + PostgreSQL**  
✅ Consome mensagens da **fila SQS** automaticamente  
✅ Publica mensagens no **SNS**  
✅ API REST para enviar notificações  

🔹 **Próximos Passos:**  
🔹 **Criar um Worker Assíncrono** para processar notificações em background  
🔹 **Adicionar autenticação (JWT) para proteger a API**  
🔹 **Deploy na AWS com ECS ou Lambda**  
