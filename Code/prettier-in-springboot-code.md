Sim! Você pode garantir essa consistência de nomenclatura utilizando **Checkstyle, SonarQube ou um plugin do Jacoco**, mas a maneira mais prática e rápida é configurar o **Checkstyle** no seu projeto Spring Boot.  

---  

## **📌 1️⃣ O que o Checkstyle faz?**  
O **Checkstyle** pode validar automaticamente se as variáveis seguem a convenção correta do **CamelCase** (`firstName` em vez de `first_name`).  

- 🚀 **Garante que todo código siga um padrão.**  
- 🛑 **Gera erros ou warnings se a convenção não for seguida.**  
- 🔍 **Funciona dentro do IntelliJ, Eclipse e VS Code.**  

---

## **📌 2️⃣ Como Configurar o Checkstyle no Spring Boot**
### **1️⃣ Adicione o plugin do Checkstyle no `pom.xml` (Maven)**  
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-checkstyle-plugin</artifactId>
            <version>3.1.2</version>
            <executions>
                <execution>
                    <phase>validate</phase>
                    <goals>
                        <goal>check</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <configLocation>checkstyle.xml</configLocation>
                <encoding>UTF-8</encoding>
                <consoleOutput>true</consoleOutput>
                <failOnViolation>true</failOnViolation>
            </configuration>
        </plugin>
    </plugins>
</build>
```

---

### **2️⃣ Crie o arquivo `checkstyle.xml` no diretório `src/main/resources`**
Esse arquivo contém as regras que garantirão que seus atributos sigam **CamelCase**.

📌 **Configuração básica do Checkstyle:**
```xml
<?xml version="1.0"?>
<module name="Checker">
    <module name="TreeWalker">
        <!-- Impede nomes com underscore (first_name) -->
        <module name="LocalVariableName">
            <property name="format" value="^[a-z][a-zA-Z0-9]*$"/>
        </module>

        <module name="MemberName">
            <property name="format" value="^[a-z][a-zA-Z0-9]*$"/>
        </module>
    </module>
</module>
```
✅ **Se alguém tentar usar `first_name` ao invés de `firstName`, o Checkstyle irá bloquear o build!** 🚀  

---

### **3️⃣ Rodar a verificação**
Agora você pode rodar:  
```sh
mvn checkstyle:check
```
Se houver variáveis fora do padrão **CamelCase**, o Checkstyle irá mostrar os erros no terminal.

---

## **📌 3️⃣ Alternativa: SonarLint (Para IntelliJ e VS Code)**
Se quiser **ver os erros diretamente no editor** sem precisar rodar o `mvn checkstyle:check`, você pode usar o **SonarLint**.  
1. Instale o plugin **SonarLint** no IntelliJ ou VS Code.  
2. Configure a regra `"java:S100"` que impede nomes como `first_name`.  

---

## **🚀 Resultado Final**
Agora, toda a equipe será forçada a usar **nomes consistentes** (`firstName`, `emailAddress`, etc.), garantindo um código mais limpo e padronizado. 🔥