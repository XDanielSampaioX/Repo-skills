# 🧠 Skill: Implementar autenticação com Google OAuth2 (@criar_auth_google)

## 🎯 Objetivo
Implementar autenticação com Google OAuth2 em um projeto Spring Boot, integrando com JWT da aplicação.

---

## 🧾 Input esperado

Crie autenticação Google @criar_auth_google

---

## 📦 Resultado esperado

1. Configuração OAuth2 Google
2. Endpoint de login com Google
3. Callback de autenticação
4. Criação/atualização de usuário
5. Geração de JWT próprio
6. Redirecionamento com token

---

## 🔧 Dependências

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

---

## ⚙️ application.yml

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: SEU_CLIENT_ID
            client-secret: SEU_CLIENT_SECRET
            scope:
              - email
              - profile
```

---

## 🔁 Fluxo

1. Usuário acessa `/oauth2/authorization/google`
2. Google autentica usuário
3. Callback retorna para aplicação
4. Extrair dados (email, nome, foto)
5. Criar ou atualizar usuário no banco
6. Gerar JWT da aplicação
7. Redirecionar frontend com token

---

## 🧩 Componentes

### OAuth2SuccessHandler

Responsável por:

- capturar dados do usuário
- gerar JWT
- redirecionar

---

### Exemplo

```java
@Component
@RequiredArgsConstructor
public class OAuth2SuccessHandler implements AuthenticationSuccessHandler {

    private final JwtService jwtService;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request,
                                        HttpServletResponse response,
                                        Authentication authentication) throws IOException {

        OAuth2User user = (OAuth2User) authentication.getPrincipal();

        String email = user.getAttribute("email");

        String token = jwtService.gerarToken(email);

        response.sendRedirect("http://localhost:3000/oauth2/success?token=" + token);
    }
}
```

---

### SecurityConfig (trecho)

```java
.oauth2Login(oauth -> oauth
    .successHandler(oAuth2SuccessHandler)
)
```

---

## 🔐 Regras obrigatórias

- Não confiar apenas no Google → sempre gerar JWT próprio
- Persistir usuário local
- Nunca usar token do Google diretamente na API
- Validar email retornado

---

## 🚫 Anti-patterns

- usar access_token do Google como autenticação da API
- não persistir usuário
- não validar retorno do OAuth
- misturar OAuth com login local sem controle

---

## 💡 Regra de ouro

> "OAuth autentica identidade. Sua aplicação controla acesso."
