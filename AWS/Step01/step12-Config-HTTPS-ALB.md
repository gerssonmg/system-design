
O que acha? Quer avançar para **HTTPS e segurança** agora? 🔐🚀

> Sim

Ótimo! Agora vamos configurar **HTTPS no Load Balancer (ALB)** para garantir a segurança da comunicação. Isso será feito com um **certificado SSL via AWS Certificate Manager (ACM)**.  

---

# **📌 Passo 12: Configurar HTTPS no ALB**

Queremos que nosso ALB aceite requisições **somente via HTTPS** e redirecione qualquer tráfego HTTP para HTTPS.

---

## **1️⃣ Criar um Certificado SSL (ACM)**
Primeiro, precisamos criar um certificado SSL no **AWS Certificate Manager (ACM)**.

### **A) Solicitar um Certificado**
Execute o seguinte comando:
```bash
aws acm request-certificate \
  --domain-name "seu-dominio.com" \
  --validation-method DNS \
  --region us-east-1
```
📌 **Substitua** `"seu-dominio.com"` pelo domínio real da sua aplicação.

### **B) Validar o Certificado**
1. Execute este comando para pegar o registro DNS que precisa ser adicionado:
   ```bash
   aws acm describe-certificate --certificate-arn <CERTIFICATE_ARN>
   ```
2. Vá até **Route 53** (se estiver usando AWS para gerenciar o domínio) e adicione o registro **CNAME** na sua zona DNS.  

3. Após alguns minutos, valide a certificação com:
   ```bash
   aws acm describe-certificate --certificate-arn <CERTIFICATE_ARN>
   ```
   **Status esperado:** `ISSUED` ✅  

---

## **2️⃣ Criar um Listener HTTPS no ALB**
Agora que temos o certificado, podemos configurar nosso **Load Balancer (ALB)** para aceitar conexões HTTPS.

```bash
aws elbv2 create-listener \
  --load-balancer-arn <ALB_ARN> \
  --protocol HTTPS --port 443 \
  --certificates CertificateArn=<CERTIFICATE_ARN> \
  --default-actions Type=forward,TargetGroupArn=<TARGET_GROUP_ARN>
```
📌 **Substitua:**  
🔹 `<ALB_ARN>` pelo ARN do seu **Application Load Balancer**  
🔹 `<CERTIFICATE_ARN>` pelo ARN do **certificado SSL emitido pelo ACM**  
🔹 `<TARGET_GROUP_ARN>` pelo ARN do **Target Group** do ECS  

---

## **3️⃣ Redirecionar HTTP para HTTPS**
Queremos que qualquer requisição HTTP seja automaticamente **redirecionada para HTTPS**.

```bash
aws elbv2 create-listener \
  --load-balancer-arn <ALB_ARN> \
  --protocol HTTP --port 80 \
  --default-actions Type=redirect,RedirectConfig='{"Protocol":"HTTPS","Port":"443","StatusCode":"HTTP_301"}'
```
Agora, qualquer requisição HTTP será **redirecionada** para HTTPS automaticamente. ✅  

---

## **4️⃣ Testar HTTPS**
Agora, tente acessar sua aplicação via HTTPS:  
```bash
curl -I https://seu-dominio.com
```
Se o certificado estiver correto, você verá uma resposta **200 OK** via HTTPS.

Caso ainda não tenha um domínio, pode testar com o DNS do ALB:  
```bash
aws elbv2 describe-load-balancers --names notification-alb
```
Isso retornará um campo `"DNSName"`, como:
```
notification-alb-123456789.us-east-1.elb.amazonaws.com
```
Teste com:
```bash
curl -I https://notification-alb-123456789.us-east-1.elb.amazonaws.com
```

---

# **🚀 Conclusão**
Agora temos um serviço **seguro e protegido com HTTPS**! 🔐🚀  
✅ **Certificado SSL válido via AWS ACM**  
✅ **Listener HTTPS configurado no ALB**  
✅ **Redirecionamento HTTP → HTTPS**  

💡 **Próximos Passos:**  
🔹 **Configurar AWS WAF** para proteção contra ataques  
🔹 **Criar alarms no CloudWatch para monitoramento de tráfego**  
🔹 **Otimizar custos ajustando configurações do ALB**  

Quer avançar para **segurança com AWS WAF** agora? 🔐🚀