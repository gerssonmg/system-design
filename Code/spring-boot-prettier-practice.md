Aqui está um exemplo completo usando **Spring Boot + JPA** com as boas práticas de nomenclatura de **Classes, Tabelas, Colunas e Relacionamentos**.

---

## **1️⃣ Entidades (Classes no Singular)**
- **User** (representa um único usuário)
- **Role** (representa um único papel de usuário)
- **UserRole** (tabela intermediária para o relacionamento N:N entre `users` e `roles`)

📌 **Usamos anotações do JPA para mapear corretamente as tabelas.**

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import java.util.Set;

@Entity
@Table(name = "users")  // ✅ Nome da tabela no plural
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "first_name", nullable = false)  // ✅ Nome de coluna no singular
    private String firstName;

    @Column(name = "last_name", nullable = false)
    private String lastName;

    @Column(name = "email", unique = true, nullable = false)
    private String email;

    @ManyToMany
    @JoinTable(
        name = "users_roles",  // ✅ Nome da tabela intermediária no plural
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles;

    // Getters e Setters
}
```

---

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import java.util.Set;

@Entity
@Table(name = "roles")  // ✅ Nome da tabela no plural
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "role_name", nullable = false, unique = true)  // ✅ Nome de coluna no singular
    private String roleName;

    @ManyToMany(mappedBy = "roles")
    private Set<User> users;

    // Getters e Setters
}
```

---

## **2️⃣ Repositórios (Padrão de nomenclatura)**
- **UserRepository** → Gerencia a entidade `User`
- **RoleRepository** → Gerencia a entidade `Role`

```java
package com.example.demo.repository;

import com.example.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> { }
```

```java
package com.example.demo.repository;

import com.example.demo.entity.Role;
import org.springframework.data.jpa.repository.JpaRepository;

public interface RoleRepository extends JpaRepository<Role, Long> { }
```

---

## **3️⃣ Serviço (UserService)**
📌 **Aqui temos uma camada de serviço para gerenciar os usuários.**
```java
package com.example.demo.service;

import com.example.demo.entity.User;
import com.example.demo.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public User getUserById(Long id) {
        return userRepository.findById(id).orElse(null);
    }

    public User saveUser(User user) {
        return userRepository.save(user);
    }
}
```

---

## **4️⃣ Controller (UserController)**
📌 **Criação do endpoint REST para gerenciar usuários.**
```java
package com.example.demo.controller;

import com.example.demo.entity.User;
import com.example.demo.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/users")  // ✅ Nome no plural para indicar coleção de recursos
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }

    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        return userService.getUserById(id);
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.saveUser(user);
    }
}
```

---

## **📌 5️⃣ Resumo e Explicação**
| **Item**          | **Padrão Usado**  | **Explicação** |
|-------------------|-----------------|---------------|
| **Classe** | `User`, `Role` | Sempre no **singular** porque representam uma única entidade. |
| **Tabela** | `users`, `roles`, `users_roles` | Sempre no **plural** porque armazenam múltiplos registros. |
| **Coluna** | `first_name`, `email` | Sempre no **singular** porque representam um único atributo. |
| **Endpoint** | `/users` | Sempre no **plural** porque retornam uma coleção de dados. |

🚀 **Com essa abordagem, seu código segue as melhores práticas de nomeação e organização.**