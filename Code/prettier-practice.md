### **Boas práticas para nomear Classes e Tabelas (Nível Senior Software Engineer)**

#### ✅ **1. Classes devem ser nomeadas no singular (PascalCase)**  
➡ **Motivo:** Classes representam um único objeto (instância), então a nomenclatura deve ser singular.  

📌 **Exemplo correto:**
```java
public class User { }  // ✅ Representa um único usuário
public class Product { }  // ✅ Representa um único produto
```

📌 **Evite:**
```java
public class Users { }  // ❌ Parece uma coleção de usuários, não um único objeto
public class Products { }  // ❌ Nome plural confunde o conceito de classe
```

---

#### ✅ **2. Tabelas no banco de dados devem ser nomeadas no plural (snake_case ou kebab-case)**  
➡ **Motivo:** Uma tabela armazena múltiplos registros (ex: vários usuários), então o nome deve ser plural.  

📌 **Exemplo correto:**
```sql
CREATE TABLE users (...);  -- ✅ Representa um conjunto de usuários
CREATE TABLE products (...);  -- ✅ Representa um conjunto de produtos
```

📌 **Evite:**
```sql
CREATE TABLE user (...);  -- ❌ Pode gerar confusão, pois "user" é um nome reservado em alguns bancos
CREATE TABLE product (...);  -- ❌ Parece que armazena apenas um produto
```

---

#### ✅ **3. Nome de colunas devem ser descritivos e no singular**
➡ **Motivo:** Cada coluna representa um único atributo.  

📌 **Exemplo correto:**
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),  -- ✅ Singular porque representa um único nome
    last_name VARCHAR(50),
    email VARCHAR(100)
);
```

📌 **Evite:**
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    names VARCHAR(50),  -- ❌ "names" sugere que pode armazenar múltiplos nomes, o que não é verdade
    emails VARCHAR(100)  -- ❌ Ambíguo, sugere múltiplos e-mails por usuário
);
```

---

#### ✅ **4. Relações e tabelas intermediárias devem seguir convenções claras**
- **1:1 → Nome baseado nas entidades envolvidas**
  ```sql
  CREATE TABLE user_profiles (...);
  ```
- **1:N → A tabela do lado N fica no plural**
  ```sql
  CREATE TABLE orders (...);
  CREATE TABLE order_items (...);
  ```
- **N:N → Nome baseado nas entidades envolvidas**
  ```sql
  CREATE TABLE user_roles (...);
  CREATE TABLE product_categories (...);
  ```

📌 **Exemplo correto:**
```sql
CREATE TABLE users_roles (
    user_id INT REFERENCES users(id),
    role_id INT REFERENCES roles(id)
);
```

---

### **Resumo Final (TL;DR)**
✅ **Classe:** Nome no **singular** (PascalCase) → `User`, `Order`, `Product`  
✅ **Tabela:** Nome no **plural** (snake_case ou kebab-case) → `users`, `orders`, `products`  
✅ **Coluna:** Nome no **singular** e descritivo → `first_name`, `email`, `price`  
✅ **Relacionamento N:N:** Nome baseado nas entidades → `users_roles`, `products_categories`  

📌 **Seguindo esse padrão, seu código e banco de dados ficarão mais organizados e fáceis de manter.** 🚀