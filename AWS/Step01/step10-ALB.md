
O que acha? Quer avançar para o Load Balancer e Auto Scaling? 🚀

> Sim

Ótimo! Agora vamos configurar o **Application Load Balancer (ALB)** e o **Auto Scaling** para garantir que nosso serviço seja escalável e acessível.  

---

# **📌 Passo 7: Criar um Load Balancer (ALB)**
Queremos distribuir o tráfego de entrada para nossas tasks no ECS.

### 1️⃣ Criar um ALB:
```bash
aws elbv2 create-load-balancer --name notification-alb \
  --subnets <SUBNET_1> <SUBNET_2> \
  --security-groups <SECURITY_GROUP_ID> \
  --scheme internet-facing \
  --type application
```
📌 **Importante:**  
- Use **subnets públicas** para um ALB voltado à internet.  
- Se o ALB for interno, configure a opção `--scheme internal`.

### 2️⃣ Criar um Target Group:
```bash
aws elbv2 create-target-group --name notification-target-group \
  --protocol HTTP \
  --port 8080 \
  --target-type ip \
  --vpc-id <VPC_ID>
```
📌 **Explicação:**  
- Como estamos usando **ECS Fargate**, precisamos do **target-type "ip"**.

### 3️⃣ Criar uma Regra de Escuta (Listener) no ALB:
```bash
aws elbv2 create-listener --load-balancer-arn <ALB_ARN> \
  --protocol HTTP --port 80 \
  --default-actions Type=forward,TargetGroupArn=<TARGET_GROUP_ARN>
```

Agora, qualquer requisição ao **ALB** será encaminhada ao serviço no ECS.

---

# **📌 Passo 8: Configurar Auto Scaling para o ECS**
Queremos que nosso serviço **escale automaticamente** de acordo com a demanda.

### 1️⃣ Criar um Auto Scaling Policy:
```bash
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --scalable-dimension ecs:service:DesiredCount \
  --resource-id service/notification-cluster/notification-service \
  --min-capacity 1 \
  --max-capacity 5
```
📌 **Explicação:**  
- Definimos que o ECS pode escalar de **1 até 5 instâncias**.

### 2️⃣ Criar uma Regra de Auto Scaling:
```bash
aws application-autoscaling put-scaling-policy \
  --service-namespace ecs \
  --scalable-dimension ecs:service:DesiredCount \
  --resource-id service/notification-cluster/notification-service \
  --policy-name cpu-auto-scaling \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration file://scaling-policy.json
```

📌 **Arquivo `scaling-policy.json`**:
```json
{
  "TargetValue": 50.0,
  "PredefinedMetricSpecification": {
    "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
  },
  "ScaleInCooldown": 60,
  "ScaleOutCooldown": 60
}
```
📌 **Explicação:**  
- Se o uso de **CPU for maior que 50%**, o ECS cria **mais tasks**.  
- Se o uso de **CPU for menor que 50%**, ele **reduz as tasks**.  

---

# **📌 Passo 9: Testar o ALB e Auto Scaling**
Agora, podemos testar o **balanceamento de carga e o escalonamento automático**.

### **1️⃣ Obtenha o DNS do ALB:**
```bash
aws elbv2 describe-load-balancers --names notification-alb
```
Você verá um **"DNSName"**, como:
```
notification-alb-123456789.us-east-1.elb.amazonaws.com
```

### **2️⃣ Teste acessando a API:**
```bash
curl -X POST "http://notification-alb-123456789.us-east-1.elb.amazonaws.com/notifications?message=HelloAWS"
```
Se a API responder, **o Load Balancer está funcionando!** ✅

### **3️⃣ Simule Auto Scaling:**
Para testar, você pode **gerar carga artificial** com Apache Bench:
```bash
ab -n 1000 -c 50 "http://notification-alb-123456789.us-east-1.elb.amazonaws.com/notifications?message=HelloAWS"
```
Depois, verifique se novas tasks ECS foram criadas:
```bash
aws ecs list-tasks --cluster notification-cluster
```

---

# **🚀 Conclusão**
Agora temos um serviço **escalável e altamente disponível** na AWS!  
✅ **Load Balancer (ALB) distribuindo as requisições**  
✅ **ECS Auto Scaling ajustando a capacidade conforme a demanda**  
✅ **Alta disponibilidade e performance otimizadas**  

💡 **Próximos Passos:**  
🔹 Configurar HTTPS no ALB com um **certificado SSL (ACM)**  
🔹 Criar um **CI/CD com GitHub Actions** para deploy automático  
🔹 Monitorar logs e métricas com **CloudWatch**  
