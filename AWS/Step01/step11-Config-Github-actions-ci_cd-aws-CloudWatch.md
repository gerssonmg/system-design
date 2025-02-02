
O que acha? Quer avançar para CI/CD ou monitoramento agora? 🚀

> Sim

Ótimo! Agora vamos configurar **CI/CD com GitHub Actions** para automatizar o deploy do nosso serviço na AWS ECS.  

---

# **📌 Passo 10: Configurar GitHub Actions para CI/CD**
Nosso objetivo é:  
✅ **Buildar o projeto**  
✅ **Gerar e enviar a imagem Docker para o ECR**  
✅ **Atualizar a task no ECS e fazer deploy automático**  

### **1️⃣ Criar um Workflow do GitHub Actions**
Crie o arquivo `.github/workflows/deploy.yml` no seu repositório:

```yaml
name: Deploy to AWS ECS

on:
  push:
    branches:
      - main  # Gatilho para deploy ao fazer push na branch main

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: notification-service
  ECS_CLUSTER: notification-cluster
  ECS_SERVICE: notification-service
  ECS_TASK_FAMILY: notification-task
  CONTAINER_NAME: notification-service

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Configurar credenciais AWS
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Fazer login no ECR
        run: |
          aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.$AWS_REGION.amazonaws.com

      - name: Build e push da imagem Docker
        run: |
          docker build -t $ECR_REPOSITORY .
          docker tag $ECR_REPOSITORY:latest <AWS_ACCOUNT_ID>.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY:latest
          docker push <AWS_ACCOUNT_ID>.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY:latest

      - name: Atualizar definição da Task no ECS
        run: |
          TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition $ECS_TASK_FAMILY)
          NEW_TASK_DEFINITION=$(echo "$TASK_DEFINITION" | jq --arg IMAGE "<AWS_ACCOUNT_ID>.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY:latest" '.taskDefinition.containerDefinitions[0].image = $IMAGE')
          echo "$NEW_TASK_DEFINITION" > new-task-def.json
          aws ecs register-task-definition --cli-input-json file://new-task-def.json

      - name: Atualizar serviço no ECS
        run: |
          aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --force-new-deployment
```

---

### **2️⃣ Criar Secrets no GitHub**
No seu repositório do GitHub, vá até:  
📌 **Settings** → **Secrets and variables** → **Actions** → **New repository secret**  

Adicione os seguintes **secrets**:  

🔹 `AWS_ACCESS_KEY_ID`  
🔹 `AWS_SECRET_ACCESS_KEY`  
🔹 `AWS_ACCOUNT_ID` (seu ID da AWS)  

---

### **3️⃣ Testar o Deploy**
Agora, **faça um commit na branch `main`** e veja o GitHub Actions rodando:  
```bash
git add .
git commit -m "Automated deployment"
git push origin main
```
✅ O workflow do GitHub Actions irá:  
1️⃣ **Buildar e enviar a imagem Docker** para o **ECR**  
2️⃣ **Atualizar a definição da task no ECS**  
3️⃣ **Forçar um novo deployment no ECS**  

---

# **📌 Passo 11: Monitoramento com AWS CloudWatch**
Agora que temos o **deploy automatizado**, precisamos monitorar logs e métricas.

### **1️⃣ Criar um Log Group no CloudWatch**
```bash
aws logs create-log-group --log-group-name /ecs/notification-service
```

### **2️⃣ Configurar Logs no ECS Task Definition**
Edite a `task-definition.json` para incluir os logs:

```json
"logConfiguration": {
  "logDriver": "awslogs",
  "options": {
    "awslogs-group": "/ecs/notification-service",
    "awslogs-region": "us-east-1",
    "awslogs-stream-prefix": "ecs"
  }
}
```

### **3️⃣ Ver Logs no CloudWatch**
```bash
aws logs tail /ecs/notification-service --follow
```

Agora, você pode ver os logs da aplicação diretamente no AWS CloudWatch! 🚀

---

# **🚀 Conclusão**
Agora temos um **pipeline de CI/CD completo e monitoramento na AWS!**  
✅ **GitHub Actions automatizando o deploy**  
✅ **Monitoramento com CloudWatch**  
✅ **Escalabilidade garantida com ECS Auto Scaling**  

💡 **Próximos Passos:**  
🔹 Configurar **HTTPS com SSL no ALB**  
🔹 Criar **Alarms no CloudWatch para alertas**  
🔹 Otimizar custos com **EC2 Spot Instances**  
