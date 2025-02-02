```
Vamos seguir para o provisionamento em produção primeiro.
```

Ótima decisão! Vamos provisionar a infraestrutura na AWS primeiro para garantir que tudo estará pronto antes do deploy da aplicação.  

---

# **📌 Provisionamento da Infraestrutura na AWS**  
Vamos usar **AWS CLI** para criar os recursos rapidamente. Se preferir **Terraform** ou **AWS CDK**, podemos estruturar isso depois.  

---

## **1️⃣ Criar Fila SQS (`notification-queue`)**  
```bash
aws sqs create-queue --queue-name notification-queue
```
### **Obter a URL da Fila**
```bash
aws sqs get-queue-url --queue-name notification-queue
```
Anote a URL, pois usaremos na aplicação.

---

## **2️⃣ Criar Tópico SNS (`notification-topic`)**  
```bash
aws sns create-topic --name notification-topic
```
### **Obter o ARN do SNS**
```bash
aws sns list-topics
```
Anote o ARN do tópico.

---

## **3️⃣ Criar Assinatura para SNS**  
Agora vamos configurar quem receberá as notificações (e-mail, SMS, etc.).  

### **Assinar um e-mail**  
```bash
aws sns subscribe \
    --topic-arn "arn:aws:sns:REGIÃO:ID_DO_SEU_CONTA:notification-topic" \
    --protocol email \
    --notification-endpoint "seuemail@exemplo.com"
```
📌 **Importante:** Você receberá um e-mail para **confirmar a assinatura**.  

### **Assinar um número de telefone (SMS)**  
```bash
aws sns subscribe \
    --topic-arn "arn:aws:sns:REGIÃO:ID_DO_SEU_CONTA:notification-topic" \
    --protocol sms \
    --notification-endpoint "+5511999999999"
```
**⚠️ Limitação:** Para contas **Free Tier**, o SNS pode não permitir envio de SMS.  

---

## **4️⃣ Criar Banco de Dados PostgreSQL no AWS RDS**  
```bash
aws rds create-db-instance \
    --db-instance-identifier notification-db \
    --engine postgres \
    --master-username admin \
    --master-user-password MinhaSenha123 \
    --allocated-storage 20 \
    --db-instance-class db.t3.micro \
    --publicly-accessible \
    --backup-retention-period 7
```
### **Obter o Endpoint do Banco**
```bash
aws rds describe-db-instances \
    --query "DBInstances[*].Endpoint.Address" \
    --output text
```
Anote o **endpoint** para configurar na aplicação.  

---

## **5️⃣ Criar Role IAM para Aplicação**  
Precisamos dar permissão para o serviço acessar **SQS e SNS**.  

```bash
aws iam create-role \
    --role-name NotificationServiceRole \
    --assume-role-policy-document file://trust-policy.json
```

Criar o arquivo `trust-policy.json`:  
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": { "Service": "ecs-tasks.amazonaws.com" },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
Agora, anexamos permissões de acesso:  
```bash
aws iam attach-role-policy \
    --role-name NotificationServiceRole \
    --policy-arn arn:aws:iam::aws:policy/AmazonSQSFullAccess

aws iam attach-role-policy \
    --role-name NotificationServiceRole \
    --policy-arn arn:aws:iam::aws:policy/AmazonSNSFullAccess
```

---

# **✅ Infraestrutura provisionada!**
Agora temos:  
✅ **SQS (`notification-queue`)**  
✅ **SNS (`notification-topic`)**  
✅ **PostgreSQL (`notification-db`)**  
✅ **Role IAM (`NotificationServiceRole`)**  

---

## **🚀 Próximo Passo**
Agora configuramos o **Spring Boot** para apontar para esses recursos e fazemos o **deploy da aplicação**. Podemos seguir com:  
🔹 **Configuração do Spring Boot para AWS**  
🔹 **Dockerização e Deploy no ECS Fargate**  

**Qual caminho prefere seguir agora?**