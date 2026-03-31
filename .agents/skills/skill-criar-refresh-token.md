# 🧠 Skill: Implementar Refresh Token (@criar_refresh_token)

## 🎯 Objetivo
Implementar sistema de refresh token para renovar o JWT sem exigir novo login do usuário.

---

## 🧾 Input esperado

Crie refresh token @criar_refresh_token

---

## 📦 O que a skill deve gerar

1. Entidade de Refresh Token
2. Endpoint para renovar token
3. Integração com JWT
4. Armazenamento seguro
5. Validação e expiração
6. Revogação de token

---

## 🔁 Fluxo de funcionamento

1. Usuário faz login → recebe:
   - access_token (curta duração)
   - refresh_token (longa duração)

2. Access token expira

3. Front envia refresh_token para:
   `/auth/refresh`

4. Sistema valida refresh_token

5. Gera novo access_token

6. Usuário continua logado sem precisar autenticar novamente

---

## 📌 Diferença entre tokens

| Tipo           | Duração       | Uso                        |
|----------------|--------------|----------------------------|
| Access Token   | Curta (min)  | Acessar API                |
| Refresh Token  | Longa (dias) | Gerar novo access token    |

---

## 🏗️ Exemplo de entidade

```java
public class RefreshToken {

    private String token;
    private String email;
    private LocalDateTime expiracao;
}
```

---

## 🔐 Endpoint

```java
@PostMapping("/refresh")
public TokenResponse refresh(@RequestBody RefreshRequest request) {
    return refreshService.renovarToken(request.getRefreshToken());
}
```

---

## 📌 Regras obrigatórias

- Refresh token deve ser armazenado (DB ou cache)
- Deve possuir expiração
- Deve ser invalidável (logout)
- Nunca confiar sem validação
- Deve estar vinculado ao usuário

---

## 🚫 Anti-patterns

- usar refresh token como access token
- não expirar refresh token
- não invalidar no logout
- salvar em local inseguro

---

## 🔒 Segurança

- usar UUID forte
- considerar rotação de refresh token
- armazenar hash do token (opcional avançado)
- usar HttpOnly cookie (recomendado)

---

## 💡 Regra de ouro

> "Access token acessa. Refresh token renova."
