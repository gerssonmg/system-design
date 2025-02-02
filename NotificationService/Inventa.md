Desafio:
O desafio de System design:
"Elabore uma plataforma para solução envio de notificações, onde existem aplicações client que enviam mensagens, e estas devem ser direcionadas à destinos como, por exemplo: Whatsapp, Telegram, E-mail, Facebook, SMS.
Essa plataforma deve armazenar os status de envio da mensagem, e também deve ser possível realizar agendamento de mensagens."


Esse desafio de System Design aborda a criação de uma **plataforma de envio de notificações** multi-canal, com agendamento e rastreamento de status de entrega. Abaixo está uma proposta de solução detalhada, com foco em alta escalabilidade, resiliência e flexibilidade.

---

## **1. Requisitos Funcionais e Não Funcionais**
### **Requisitos Funcionais**
- Aplicações cliente podem enviar mensagens para a plataforma.
- Suporte a múltiplos canais de envio: WhatsApp, Telegram, Email, Facebook, SMS.
- Armazenamento do status das mensagens: "Pendente", "Enviado", "Entregue", "Erro".
- Suporte ao agendamento de mensagens para envio futuro.
- APIs para consulta do status das mensagens.
- Garantia de envio, mesmo em caso de falhas no sistema.

### **Requisitos Não Funcionais**
- Alta escalabilidade para lidar com grandes volumes de mensagens.
- Latência baixa para envio de mensagens em tempo real.
- Tolerância a falhas e alta disponibilidade.
- Modularidade para adicionar novos canais no futuro.

---

## **2. Arquitetura Geral**

A arquitetura será baseada em uma abordagem orientada a eventos, com uso extensivo de **filas** e **workers** para processar mensagens de forma assíncrona.

### **Componentes Principais**
1. **API Gateway**:
   - Expondo endpoints para que os clientes enviem mensagens e consultem status.
   - Valida as requisições e autentica os clientes.

2. **Mensagem Service**:
   - Processa as mensagens recebidas do cliente.
   - Enfileira mensagens para o **Dispatcher Service**.

3. **Dispatcher Service**:
   - Consome mensagens das filas e as entrega aos provedores de canais (WhatsApp, Email, SMS, etc.).
   - Atualiza o status da mensagem no banco de dados.

4. **Scheduler Service**:
   - Gerencia mensagens agendadas e as envia para a fila no horário correto.

5. **Status Service**:
   - Permite consulta ao status das mensagens armazenadas.

6. **Storage**:
   - Banco de dados para armazenar as mensagens, incluindo informações de status, remetente, destinatário, conteúdo e metadados.

---

## **3. Fluxo de Dados**
1. O cliente faz um POST na API com os detalhes da mensagem (destinatário, canal, conteúdo, data de envio).
2. A API valida a requisição e a envia ao **Mensagem Service**.
3. O **Mensagem Service**:
   - Se for mensagem imediata, envia para a fila do **Dispatcher Service**.
   - Se for mensagem agendada, registra no banco de dados e no **Scheduler Service**.
4. O **Dispatcher Service** consome mensagens da fila:
   - Identifica o canal.
   - Chama o provedor adequado (WhatsApp API, SMTP, etc.).
   - Atualiza o status no banco de dados.
5. O cliente pode consultar o status da mensagem via **Status Service**.

---

## **4. Escolha de Tecnologias**
- **API Gateway**: NGINX ou AWS API Gateway.
- **Mensageria**: RabbitMQ, Kafka, ou SQS para filas.
- **Banco de Dados**:
  - Relacional (PostgreSQL) para metadados e status.
  - Redis para cache de mensagens e operações rápidas.
- **Armazenamento de Mensagens Agendadas**: Redis com TTL ou banco relacional.
- **Dispatcher**: Serviços específicos para cada canal (integrados via SDKs ou APIs dos provedores).

---

## **5. Design de Banco de Dados**

### **Tabela: Messages**
| **Campo**       | **Tipo**         | **Descrição**                       |
|------------------|------------------|-------------------------------------|
| id              | UUID            | Identificador único da mensagem     |
| client_id       | UUID            | ID do cliente que enviou a mensagem |
| destination     | VARCHAR(255)    | Destinatário (e.g., número de celular, email) |
| channel         | ENUM            | Canal (WhatsApp, Email, SMS, etc.)  |
| content         | TEXT            | Conteúdo da mensagem                |
| schedule_time   | TIMESTAMP       | Data/hora de envio agendado         |
| status          | ENUM            | Status da mensagem ("Pendente", "Enviado", etc.) |
| created_at      | TIMESTAMP       | Timestamp de criação                |

---

## **6. Escalabilidade e Tolerância a Falhas**

### **Escalabilidade**
- **Horizontal Scaling**:
  - Workers do **Dispatcher Service** podem ser escalados horizontalmente para lidar com picos de tráfego.
  - API Gateway pode usar um Load Balancer para distribuir as requisições.
- **Filas**: Garantem processamento assíncrono e evitam sobrecarregar os provedores de canais.

### **Tolerância a Falhas**
- **Retries**: Mensagens com falha de envio serão re-enfileiradas para tentativa posterior.
- **DLQ (Dead Letter Queue)**: Armazena mensagens que falharam após múltiplas tentativas.
- **Status Tracking**: Status detalhado para identificar problemas de entrega.

---

## **7. Detalhes sobre Agendamento**
- O **Scheduler Service** utiliza o Redis com **Sorted Sets** para mensagens agendadas.
- Mensagens com horário futuro são armazenadas com o timestamp como score.
- Um worker verifica periodicamente o Redis para identificar mensagens cujo horário chegou.

---

## **8. Possíveis Perguntas na Entrevista**
1. **Como você lidaria com falhas de provedores de canais, como o WhatsApp?**
   - Resposta: Implementaria um sistema de retries com backoff exponencial e usaria DLQs para mensagens que falham consistentemente. Além disso, notificaria os clientes em caso


de falhas irreparáveis, oferecendo alternativas, como envio por outros canais disponíveis (fallback).

---

2. **Como garantir a entrega de mensagens em tempo real e agendadas ao mesmo tempo?**
   - Resposta: Utilizaria filas separadas para mensagens em tempo real e agendadas. Mensagens agendadas seriam gerenciadas pelo **Scheduler Service**, que monitora um banco de dados ou Redis para liberar as mensagens no momento adequado, enviando-as para a fila de processamento do Dispatcher Service.

---

3. **Como você trataria mensagens duplicadas?**
   - Resposta: Implementaria um mecanismo de deduplicação utilizando um identificador único para cada mensagem (`idempotency key`). No Dispatcher Service, verificaria se o `id` da mensagem já foi processado antes de reenviá-la.

---

4. **Como armazenar mensagens com alto volume de dados, especialmente para grandes clientes?**
   - Resposta: Utilizaria sharding no banco de dados baseado no `client_id` ou `destination` para distribuir a carga. Para mensagens antigas ou arquivadas, moveria os dados para um sistema de armazenamento de longo prazo como S3 ou Glacier.

---

5. **Como você lidaria com picos de tráfego, como em Black Friday?**
   - Resposta: Escalaria horizontalmente os componentes da arquitetura (API Gateway, Dispatcher Workers, etc.). Usaria filas elásticas (ex.: Kafka ou SQS) para desacoplar produtores e consumidores, evitando sobrecarga. Implementaria limites de taxa (rate limiting) no API Gateway para proteger a infraestrutura de abusos.

---

## **9. Detalhes de Implementação (Pseudocódigo do Scheduler Service)**

```java
// SchedulerService.java
public class SchedulerService {
    private final RedisClient redisClient;
    private final QueueService queueService;

    public SchedulerService(RedisClient redisClient, QueueService queueService) {
        this.redisClient = redisClient;
        this.queueService = queueService;
    }

    public void scheduleMessage(String messageId, String payload, long scheduleTime) {
        redisClient.zAdd("scheduledMessages", scheduleTime, messageId);
        redisClient.set("message:" + messageId, payload);
    }

    public void processScheduledMessages() {
        long now = System.currentTimeMillis();
        Set<String> messagesToSend = redisClient.zRangeByScore("scheduledMessages", 0, now);

        for (String messageId : messagesToSend) {
            String payload = redisClient.get("message:" + messageId);
            queueService.sendToDispatcher(payload);
            redisClient.z
```

> Explique mais sobre o requisito de "Agendamento de Mensagens". O recrutador me questionou muito essa parte. Não sube responder

O agendamento de mensagens é um requisito importante para plataformas de envio de notificações, pois permite que os clientes definam uma data e hora específica para o envio de mensagens. Esse recurso é amplamente utilizado em campanhas de marketing, lembretes automáticos ou notificações baseadas em eventos futuros.

### **Como funciona o Agendamento de Mensagens**
1. **Recebimento do pedido de agendamento**:
   - O cliente envia uma solicitação de envio de mensagem, incluindo a data e hora desejadas para o envio.
   - Essa informação é registrada no sistema.

2. **Armazenamento das mensagens agendadas**:
   - As mensagens não são enviadas imediatamente. Em vez disso, são armazenadas em uma estrutura de dados eficiente para recuperação futura (por exemplo, banco de dados ou Redis).
   - Juntamente com o conteúdo da mensagem, armazenam-se metadados como:
     - Canal de envio (WhatsApp, Email, etc.).
     - Destinatário.
     - Data e hora de agendamento.
     - Estado atual (ex.: "Agendado").

3. **Processamento no horário certo**:
   - Um componente do sistema (Scheduler Service) monitora mensagens agendadas e as envia para processamento no momento correto.
   - Quando o horário chega, a mensagem é movida para uma fila de envio, onde será tratada como uma mensagem normal.

4. **Envio e atualização do status**:
   - O sistema entrega a mensagem ao canal apropriado.
   - O status da mensagem é atualizado no banco de dados para permitir rastreamento.

---

### **Como implementar o agendamento de mensagens**
Existem várias estratégias para implementar o agendamento. A escolha depende do volume de mensagens, da precisão necessária e da infraestrutura disponível.

#### **1. Agendamento usando banco de dados (SQL-based)**:
- **Como funciona**:
  - Mensagens agendadas são armazenadas em uma tabela com um campo `schedule_time` (timestamp).
  - Um **worker** ou **cron job** consulta periodicamente o banco de dados por mensagens cujo `schedule_time` é menor ou igual ao horário atual.
  - Essas mensagens são movidas para uma fila de envio.
  
- **Vantagens**:
  - Simples de implementar.
  - Usa tecnologias comuns (bancos de dados relacionais como PostgreSQL).
  
- **Desvantagens**:
  - Ineficiente para altíssimos volumes (o polling constante pode sobrecarregar o banco).
  - Pode haver pequenos atrasos devido ao intervalo entre consultas.

- **SQL para consulta periódica**:
  ```sql
  SELECT * 
  FROM messages 
  WHERE status = 'SCHEDULED' 
    AND schedule_time <= NOW();
  ```

---

#### **2. Agendamento usando Redis (time-based processing)**:
- **Como funciona**:
  - As mensagens são armazenadas em um **Sorted Set** do Redis.
  - A chave do Sorted Set pode ser algo como `scheduled_messages`, e o score de cada mensagem é o timestamp do agendamento.
  - Um **worker** verifica regularmente o Sorted Set para mensagens com score menor ou igual ao horário atual.

- **Vantagens**:
  - Muito eficiente para alta escala (Redis é extremamente rápido para operações baseadas em chaves e scores).
  - Baixa latência no processamento.

- **Desvantagens**:
  - Requer Redis como dependência adicional.
  - Pode não ser adequado para mensagens com exigências complexas de armazenamento.

- **Exemplo de pseudocódigo**:
  ```java
  public void processScheduledMessages() {
      long now = System.currentTimeMillis();
      Set<String> messagesToSend = redisClient.zRangeByScore("scheduledMessages", 0, now);

      for (String messageId : messagesToSend) {
          String payload = redisClient.get("message:" + messageId);
          queueService.sendToDispatcher(payload);
          redisClient.zRem("scheduledMessages", messageId); // Remove da fila de agendamento
      }
  }
  ```

---

#### **3. Agendamento usando filas e mensagens com TTL**
- **Como funciona**:
  - Mensagens agendadas são publicadas diretamente em uma fila (como RabbitMQ ou Kafka) com um atributo de **Time-To-Live (TTL)**.
  - A fila só libera a mensagem para consumo após o tempo configurado no TTL.
  
- **Vantagens**:
  - Altamente escalável.
  - Simples de configurar com mensagerias modernas.
  
- **Desvantagens**:
  - Difícil alterar mensagens agendadas (como reprogramá-las) após serem publicadas na fila.
  - Não oferece flexibilidade para lógica complexa de agendamento.

---

### **Desafios e Soluções no Agendamento**
#### **1. Precisão**
- **Problema**: A mensagem deve ser enviada exatamente no horário agendado.
- **Solução**: Use sistemas baseados em Redis ou bancos otimizados para baixa latência. Mantenha workers rodando continuamente para processar mensagens na hora exata.

#### **2. Escalabilidade**
- **Problema**: Mensagens agendadas podem crescer rapidamente, tornando o processamento ineficiente.
- **Solução**: Use uma abordagem orientada a eventos e distribua a carga usando filas distribuídas como Kafka ou RabbitMQ.

#### **3. Falhas no processamento**
- **Problema**: Mensagens agendadas podem ser perdidas devido a falhas no sistema.
- **Solução**: Implemente persistência confiável (banco de dados ou Redis persistente) e garanta retries automáticos em caso de falhas.

#### **4. Reagendamento**
- **Problema**: O cliente pode querer alterar o horário de envio de uma mensagem.
- **Solução**: Implemente APIs para que clientes possam atualizar mensagens antes do horário programado. Certifique-se de que a alteração reflita no armazenamento.

---

### **Como explicar isso em uma entrevista**
Se questionado sobre o agendamento de mensagens, você pode seguir este roteiro:
1. **Explique o conceito**:
   - "Agendar mensagens significa permitir que o cliente defina o horário exato para que a mensagem seja enviada."
   - "Isso é feito armazenando as mensagens com suas informações e processando-as no momento correto."

2. **Descreva a abordagem técnica**:
   - "Existem três abordagens comuns para agendamento: polling com banco de dados, uso de Redis com Sorted Sets e filas com TTL."
   - "A escolha depende dos requisitos do sistema, como precisão, volume e tolerância a falhas."

3. **Detalhe os desafios e como resolvê-los**:
   - "Para lidar com precisão, usamos mecanismos como Redis ou filas com TTL. Para escalabilidade, dividimos a carga em filas distribuídas e escalamos workers horizontalmente."

4. **Enfatize a robustez**:
   - "Sempre garanto que o sistema seja tolerante a falhas, utilizando mecanismos de persistência e retries automáticos."

---

Se o recrutador for rigoroso, esteja preparado para implementar um **Scheduler Service** em tempo real e discutir vantagens e desvantagens de cada abordagem.


> como lidaria com filas de alta e baixa prioridade?

Lidar com filas de alta e baixa prioridade em um sistema de mensageria envolve a implementação de mecanismos que garantam que as mensagens mais críticas (alta prioridade) sejam processadas antes das mensagens menos importantes (baixa prioridade), mesmo que ambas sejam recebidas simultaneamente.

Aqui estão os detalhes sobre como abordar esse cenário:

---

### **1. Estrutura das filas**
- **Filas separadas para cada prioridade**:
  - Crie filas distintas para mensagens de alta e baixa prioridade. Por exemplo:
    - `high-priority-queue`
    - `low-priority-queue`
  - Isso permite que você processe mensagens de alta prioridade primeiro, mesmo que haja mensagens na fila de baixa prioridade.

- **Exemplo com RabbitMQ**:
  - Configure duas filas, e consumidores diferentes podem ser ajustados para processar as mensagens:
    - Um consumidor com mais threads para a fila de alta prioridade.
    - Outro consumidor com menos threads para a fila de baixa prioridade.

---

### **2. Consumidores ajustados por prioridade**
- Dê **maior capacidade de processamento** às mensagens de alta prioridade. Por exemplo:
  - Crie **10 consumidores** para a fila de alta prioridade.
  - Crie **2 consumidores** para a fila de baixa prioridade.
- Isso garante que a fila de alta prioridade seja drenada mais rapidamente.

---

### **3. Implementação com **Redis** (Sorted Sets)**
- Use um **Sorted Set**, onde o score determina a prioridade:
  - Mensagens de alta prioridade recebem scores baixos.
  - Mensagens de baixa prioridade recebem scores altos.
- Exemplo:
  - Score 1 para alta prioridade.
  - Score 100 para baixa prioridade.

**Processamento:**
- Os consumidores sempre buscam mensagens com os menores scores primeiro:
  ```java
  // Processa as mensagens de menor prioridade primeiro
  Set<String> messages = redisClient.zRangeByScore("messageQueue", 0, currentTimeMillis());
  ```

---

### **4. Usando filas com prioridades integradas**
Algumas ferramentas de mensageria, como **RabbitMQ** e **ActiveMQ**, suportam nativamente **prioridades de mensagem**.

- **Como funciona no RabbitMQ**:
  - Configure a fila para aceitar mensagens com prioridade:
    ```bash
    channel.queueDeclare("priority-queue", durable, exclusive, autoDelete, { "x-max-priority": 10 });
    ```
  - Mensagens podem ser enviadas com um atributo de prioridade (0-10):
    ```java
    AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
      .priority(9)
      .build();
    channel.basicPublish("", "priority-queue", props, messageBody);
    ```
  - O RabbitMQ entrega automaticamente as mensagens de maior prioridade primeiro.

---

### **5. Estratégia com Kafka (não possui prioridade nativa)**
- Kafka não tem suporte nativo para prioridades, mas você pode simular isso:
  - **Criação de tópicos separados**:
    - Crie dois tópicos: `high-priority` e `low-priority`.
    - Configure consumidores para priorizar o consumo do tópico `high-priority`.
  - **Partições baseadas em prioridade**:
    - Envie mensagens de alta prioridade para as primeiras partições e configure consumidores para priorizá-las.

---

### **6. Solução baseada em banco de dados**
- Use um banco de dados para armazenar as mensagens com prioridade associada.
- Consuma as mensagens ordenadas pela prioridade:
  - Alta prioridade primeiro, depois baixa prioridade.
  ```sql
  SELECT * 
  FROM messages 
  WHERE status = 'PENDING'
  ORDER BY priority ASC, created_at ASC
  LIMIT 100;
  ```
---

### **Desafios e Soluções**
1. **Desbalanceamento entre filas**:
   - **Problema**: A fila de baixa prioridade pode crescer muito, atrasando mensagens.
   - **Solução**: Garanta consumidores mínimos para a fila de baixa prioridade, mesmo durante picos na alta prioridade.

2. **Starvation** (fome de mensagens de baixa prioridade):
   - **Problema**: Mensagens de baixa prioridade podem nunca ser processadas.
   - **Solução**: A cada N mensagens de alta prioridade, processe uma de baixa prioridade.

3. **Processamento em tempo real**:
   - **Problema**: Mensagens de alta prioridade podem ser críticas (como alertas), enquanto as de baixa prioridade podem ser notificações menos urgentes.
   - **Solução**: Implemente monitoramento para garantir que mensagens de alta prioridade sejam drenadas rapidamente, e adicione um SLA.

---

### **Exemplo em Pseudocódigo**
```java
public void processMessages() {
    while (true) {
        // Tente buscar da fila de alta prioridade primeiro
        Message message = highPriorityQueue.poll();
        
        if (message == null) {
            // Se não houver mensagens na alta, busque na baixa
            message = lowPriorityQueue.poll();
        }
        
        if (message != null) {
            process(message);
        } else {
            // Aguarde antes de tentar novamente
            Thread.sleep(100);
        }
    }
}
```

---

### **Resumo**
- Para lidar com filas de alta e baixa prioridade, escolha a estratégia com base nas ferramentas disponíveis:
  - Use filas separadas para maior controle.
  - Aproveite o suporte nativo de prioridades, se possível.
  - Implemente starvation prevention para mensagens de baixa prioridade.
- Certifique-se de monitorar o desempenho do sistema e ajustar a capacidade de processamento conforme necessário.