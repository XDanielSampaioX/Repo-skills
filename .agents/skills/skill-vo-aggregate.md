# 🧠 Skill: Criar objeto composto por Value Objects (@vo)

## 🎯 Objetivo
Criar uma classe de domínio composta por múltiplos Value Objects, garantindo validação forte e encapsulamento.

---

## 🧾 Input esperado

Crie um @vo Pessoa

---

## 📌 Regras obrigatórias

1. A classe principal:
   - NÃO deve usar prefixo `vo_`
   - Representa um conceito de domínio (ex: Pessoa, Produto, Endereco)

2. Os atributos:
   - Devem ser Value Objects
   - Cada atributo deve ter sua própria classe `vo_`

3. A classe deve:
   - Ser imutável
   - Validar consistência entre os VOs
   - Não permitir estado inválido

---

## 🏗️ Exemplo completo

### Classe principal

```java
import lombok.Getter;
import java.util.Objects;

@Getter
public class Pessoa {

    private final vo_Nome nome;
    private final vo_UserEmail email;

    public Pessoa(vo_Nome nome, vo_UserEmail email) {
        validar(nome, email);
        this.nome = nome;
        this.email = email;
    }

    private void validar(vo_Nome nome, vo_UserEmail email) {
        if (nome == null) {
            throw new IllegalArgumentException("Nome é obrigatório");
        }

        if (email == null) {
            throw new IllegalArgumentException("Email é obrigatório");
        }
    }

    public String getPrimeiroNome() {
        return nome.getPrimeiroNome();
    }

    public boolean isEmailCorporativo() {
        return email.isCorporativo();
    }

    public boolean mesmoDominio(Pessoa outraPessoa) {
        return this.email.mesmoDominio(outraPessoa.getEmail());
    }

    public boolean isIgual(Pessoa outraPessoa) {
        return this.equals(outraPessoa);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Pessoa pessoa)) return false;
        return Objects.equals(nome, pessoa.nome) &&
               Objects.equals(email, pessoa.email);
    }

    @Override
    public int hashCode() {
        return Objects.hash(nome, email);
    }
}
```

---

### VO Nome

```java
import lombok.Getter;
import java.util.Objects;

@Getter
public class vo_Nome {

    private final String value;

    public vo_Nome(String value) {
        validar(value);
        this.value = value.trim();
    }

    private void validar(String value) {
        if (value == null || value.isBlank()) {
            throw new IllegalArgumentException("Nome não pode ser vazio");
        }
    }

    public String getPrimeiroNome() {
        return value.split(" ")[0];
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof vo_Nome that)) return false;
        return Objects.equals(value, that.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
}
```

---

### VO Email

```java
import lombok.Getter;
import java.util.Objects;
import java.util.regex.Pattern;

@Getter
public class vo_UserEmail {

    private static final Pattern EMAIL_PATTERN =
            Pattern.compile("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$");

    private final String value;

    public vo_UserEmail(String value) {
        validar(value);
        this.value = value.trim().toLowerCase();
    }

    private void validar(String value) {
        if (value == null || value.isBlank()) {
            throw new IllegalArgumentException("Email não pode ser vazio");
        }

        if (!EMAIL_PATTERN.matcher(value).matches()) {
            throw new IllegalArgumentException("Email inválido");
        }
    }

    public String getDominio() {
        return value.substring(value.indexOf("@") + 1);
    }

    public boolean isCorporativo() {
        return !getDominio().equals("gmail.com");
    }

    public boolean mesmoDominio(vo_UserEmail outro) {
        return this.getDominio().equals(outro.getDominio());
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof vo_UserEmail that)) return false;
        return Objects.equals(value, that.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
}
```

---

## 🧠 Regra de ouro

> "Entidade usa VO. VO protege o domínio."
