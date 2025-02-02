```
Ok. Vamos executar localmente 
```

Ótima escolha! Executar localmente é **mais rápido** e **flexível** para um MVP.  

Agora, vamos **rodar o Terraform localmente** para provisionar a infraestrutura na AWS.

---

## **📌 Passo a Passo para Executar o Terraform Localmente**

### **1️⃣ Instalar Terraform**
Se ainda não tiver o Terraform instalado:  
- **Linux/macOS:**  
  ```bash
  brew install terraform
  ```
- **Windows:**  
  Baixe o binário oficial: [terraform.io/downloads](https://developer.hashicorp.com/terraform/downloads)  

Verifique a instalação:  
```bash
terraform version
```

---

### **2️⃣ Configurar Credenciais da AWS**
Terraform precisa de acesso à AWS. Configure as credenciais no terminal:  

```bash
aws configure
```
📌 Informe:  
- **AWS Access Key ID**  
- **AWS Secret Access Key**  
- **Região**: `us-east-1` (ou outra)  

Teste se está funcionando:  
```bash
aws sts get-caller-identity
```
Se retornar os detalhes da conta, está pronto para uso.

---

### **3️⃣ Criar Diretório do Projeto**
Se ainda não criou, organize assim:

```bash
mkdir -p notification-service/infra && cd notification-service/infra
```

---

### **4️⃣ Criar Arquivo `main.tf`**
Este arquivo provisiona a **fila SQS, o tópico SNS e o banco PostgreSQL (RDS)**.

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_sqs_queue" "notification_queue" {
  name                      = "notification-queue"
  delay_seconds             = 0
  visibility_timeout_seconds = 30
  message_retention_seconds = 345600
}

resource "aws_sns_topic" "notification_topic" {
  name = "notification-topic"
}

resource "aws_db_instance" "notification_db" {
  identifier             = "notification-db"
  engine                = "postgres"
  instance_class        = "db.t3.micro"
  allocated_storage     = 20
  username             = "admin"
  password             = "MinhaSenha123"
  publicly_accessible  = true
  skip_final_snapshot  = true
}
```

---

### **5️⃣ Inicializar o Terraform**
Dentro do diretório `infra/`, rode:

```bash
terraform init
```
📌 Isso baixa os plugins necessários.

---

### **6️⃣ Verificar o Plano de Execução**
Antes de criar os recursos, veja o que será provisionado:

```bash
terraform plan
```
📌 O Terraform mostra as mudanças **sem aplicar** nada.

---

### **7️⃣ Aplicar as Mudanças**
Se tudo estiver certo, rode:

```bash
terraform apply -auto-approve
```
📌 Isso **cria** os recursos na AWS.

---

### **8️⃣ Confirmar que Funcionou**
Após a execução, verifique os serviços na AWS:  
- 📌 **SQS**: Vá para **AWS Console → SQS**  
- 📌 **SNS**: Vá para **AWS Console → SNS**  
- 📌 **RDS**: Vá para **AWS Console → RDS**  

Se tudo estiver lá, **infraestrutura provisionada com sucesso!** 🎉

---

### **🚀 Próximos Passos**
Agora que temos a infraestrutura pronta, podemos:  
✅ **Configurar o Spring Boot** para usar o SQS, SNS e PostgreSQL.  
✅ **Criar um Producer e Consumer para o SQS**.  
✅ **Testar o fluxo de mensagens no serviço de Notificação**.  
