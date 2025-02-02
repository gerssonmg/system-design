
Gostou desse caminho? Quer que avancemos para o deploy? 🚀


> Sim

Ótimo! Agora vamos fazer o **deploy na AWS**.  

📌 **Cenário:** Queremos rodar nosso serviço **Spring Boot** em produção.  
💡 **Solução:** Vamos usar **AWS ECS (Fargate) + RDS + SQS + SNS**.

---

## **📌 Passo 1: Criar a Imagem Docker**
Dentro do diretório do projeto, crie um **Dockerfile**:

```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/notification-service.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Agora, crie a imagem:

```bash
mvn clean package -DskipTests
docker build -t notification-service .
```

Teste localmente:

```bash
docker run -p 8080:8080 -e AWS_ACCESS_KEY_ID=XXX -e AWS_SECRET_ACCESS_KEY=XXX notification-service
```

Se rodar sem erros, podemos subir para a AWS. 🚀

---

## **📌 Passo 2: Criar um Repositório no Amazon ECR**
Vamos armazenar a imagem no **Elastic Container Registry (ECR)**:

1️⃣ Crie um repositório no ECR:  
```bash
aws ecr create-repository --repository-name notification-service
```

2️⃣ Obtenha a URL do repositório:  
```bash
aws ecr describe-repositories --repository-names notification-service
```

3️⃣ Faça login no ECR:  
```bash
aws ecr get-login-password | docker login --username AWS --password-stdin <URL_DO_REPOSITORIO>
```

4️⃣ Envie a imagem:  
```bash
docker tag notification-service <URL_DO_REPOSITORIO>:latest
docker push <URL_DO_REPOSITORIO>:latest
```

Agora a imagem está na AWS! 🎉

---

## **📌 Passo 3: Criar o Cluster ECS (Fargate)**
Rodamos o serviço no **ECS Fargate**, sem precisar gerenciar servidores.

1️⃣ Criar cluster:
```bash
aws ecs create-cluster --cluster-name notification-cluster
```

2️⃣ Criar um papel IAM para permitir ao ECS acessar os serviços:
```bash
aws iam create-role --role-name ecsTaskExecutionRole \
  --assume-role-policy-document file://ecs-trust-policy.json
```

**Arquivo `ecs-trust-policy.json`:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

3️⃣ Associar permissões:
```bash
aws iam attach-role-policy --role-name ecsTaskExecutionRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```

---

## **📌 Passo 4: Criar a Tarefa ECS**
Criamos a **Task Definition** para rodar o container.

1️⃣ Criar o JSON da tarefa `task-definition.json`:
```json
{
  "family": "notification-task",
  "networkMode": "awsvpc",
  "executionRoleArn": "arn:aws:iam::ACCOUNT_ID:role/ecsTaskExecutionRole",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "notification-service",
      "image": "<URL_DO_REPOSITORIO>:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8080,
          "hostPort": 8080
        }
      ],
      "environment": [
        {"name": "SPRING_DATASOURCE_URL", "value": "jdbc:postgresql://notification-db.AWS_REGION.rds.amazonaws.com:5432/postgres"},
        {"name": "SPRING_DATASOURCE_USERNAME", "value": "admin"},
        {"name": "SPRING_DATASOURCE_PASSWORD", "value": "MinhaSenha123"}
      ]
    }
  ]
}
```

2️⃣ Criar a definição da tarefa:
```bash
aws ecs register-task-definition --cli-input-json file://task-definition.json
```

---

## **📌 Passo 5: Criar o Serviço ECS**
Criamos o serviço para rodar a aplicação continuamente.

```bash
aws ecs create-service --cluster notification-cluster \
  --service-name notification-service \
  --task-definition notification-task \
  --desired-count 1 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[SUBNET_ID],securityGroups=[SECURITY_GROUP_ID],assignPublicIp=ENABLED}"
```

Agora a aplicação está rodando no ECS! 🚀

---

## **📌 Passo 6: Testar a Aplicação**
1️⃣ Obtenha o IP da tarefa:
```bash
aws ecs describe-tasks --cluster notification-cluster --tasks TASK_ID
```

2️⃣ Teste chamando a API:
```bash
curl -X POST "http://<TASK_IP>:8080/notifications?message=HelloAWS"
```

Se funcionar, **deploy concluído com sucesso!** 🎉

---

## **🚀 Conclusão**
Agora temos nosso **serviço de notificação rodando na AWS com ECS Fargate!**  

✅ **Imagem Docker enviada para o ECR**  
✅ **Serviço rodando no ECS com PostgreSQL + SQS + SNS**  
✅ **Segurança configurada com IAM**  
✅ **Infraestrutura provisionada automaticamente**  

💡 **Próximos Passos:**  
🔹 Criar um **ALB (Load Balancer)** para distribuir o tráfego  
🔹 Configurar **Auto Scaling** para escalar dinamicamente  
🔹 Criar um **CI/CD com GitHub Actions** para automação  
