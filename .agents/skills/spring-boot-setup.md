# Skill: Spring Boot Application Setup

## Description
Cria a estrutura inicial de um projeto Spring Boot para aplicações backend Java, incluindo configuração de dependências, estrutura de pacotes e arquivo de propriedades.

## Instructions

### 1. Criar projeto via Spring Initializr
Utilize o [Spring Initializr](https://start.spring.io/) ou o comando abaixo para gerar a estrutura do projeto:
```bash
curl https://start.spring.io/starter.zip \
  -d type=maven-project \
  -d language=java \
  -d bootVersion=3.2.0 \
  -d baseDir=meu-projeto \
  -d groupId=com.exemplo \
  -d artifactId=meu-projeto \
  -d name=meu-projeto \
  -d packageName=com.exemplo.meuprojeto \
  -d packaging=jar \
  -d javaVersion=17 \
  -d dependencies=web,data-jpa,security,validation,actuator \
  -o meu-projeto.zip
unzip meu-projeto.zip
```

### 2. Estrutura de pacotes recomendada
```
src/main/java/com/exemplo/meuprojeto/
├── MeuProjetoApplication.java
├── controller/
├── service/
├── repository/
├── model/
│   ├── entity/
│   └── dto/
├── config/
└── exception/
```

### 3. Configuração do `application.properties`
```properties
# Server
server.port=8080

# Database
spring.datasource.url=jdbc:postgresql://localhost:5432/meubanco
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASS}
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# Logging
logging.level.root=INFO
logging.level.com.exemplo=DEBUG
```

### 4. Classe principal
```java
@SpringBootApplication
public class MeuProjetoApplication {
    public static void main(String[] args) {
        SpringApplication.run(MeuProjetoApplication.class, args);
    }
}
```

## Dependencies (pom.xml)
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```
