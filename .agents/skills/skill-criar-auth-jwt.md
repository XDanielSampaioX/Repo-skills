# 🧠 Skill: Implementar autenticação JWT (@criar_auth_jwt)

## 🎯 Objetivo
Implementar autenticação JWT em um projeto Java com Spring Boot, usando boas práticas de segurança, separação de responsabilidades e estrutura pronta para expansão.

---

## 🧾 Input esperado

Crie autenticação JWT @criar_auth_jwt

Exemplos:
- Crie autenticação JWT @criar_auth_jwt
- Implemente login com JWT @criar_auth_jwt
- Adicione autenticação JWT neste projeto @criar_auth_jwt

---

## 📌 Stack padrão

Esta skill deve assumir como padrão:

- Java
- Spring Boot
- Spring Security
- JWT
- Lombok
- MapStruct
- Swagger/OpenAPI

---

## 📦 Resultado esperado

A skill deve gerar ou ajustar:

1. dependências necessárias
2. configuração do Spring Security
3. endpoint de autenticação
4. DTOs de login e resposta
5. service de autenticação
6. classe utilitária/service para geração e validação do JWT
7. filtro JWT
8. classe de usuário/autenticação compatível com o projeto
9. rotas públicas e privadas
10. integração com Swagger
11. exemplo de uso
12. observações de segurança

---

## 📌 Regras obrigatórias

1. A senha deve ser armazenada com `BCryptPasswordEncoder`.
2. O token JWT deve ser gerado apenas após validação das credenciais.
3. O filtro JWT deve validar o token a cada requisição protegida.
4. O endpoint de login deve ser público.
5. Rotas privadas devem exigir autenticação.
6. O código deve separar:
   - controller
   - service
   - security
   - DTOs
7. Nunca retornar senha na resposta.
8. Nunca colocar regra de negócio de autenticação dentro do controller.
9. O Swagger deve aceitar autenticação Bearer.
10. Sempre que possível, preservar a estrutura atual do projeto.

---

## 🧩 Passo a passo de execução

### Etapa 1 — Adicionar dependências

A skill deve garantir as dependências de:

- Spring Security
- JWT
- Lombok
- MapStruct
- Swagger/OpenAPI

### Exemplo de dependências Maven

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.12.7</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.12.7</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.12.7</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

---

### Etapa 2 — Definir propriedades

A skill deve criar propriedades como:

```properties
jwt.secret=defina-uma-chave-base64-segura
jwt.expiration=86400000
```

Ou em `application.yml`:

```yaml
jwt:
  secret: defina-uma-chave-base64-segura
  expiration: 86400000
```

---

### Etapa 3 — Criar DTOs

A skill deve criar DTOs mínimos para autenticação.

#### Exemplo `AutenticacaoRequest.java`

```java
package com.example.projeto.modules.auth.domain;

import lombok.Data;

@Data
public class AutenticacaoRequest {
    private String email;
    private String senha;
}
```

#### Exemplo `AutenticacaoResponse.java`

```java
package com.example.projeto.modules.auth.domain;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;

@Data
@Builder
@AllArgsConstructor
public class AutenticacaoResponse {
    private String token;
    private String tipo;
}
```

---

### Etapa 4 — Criar serviço JWT

A skill deve criar uma classe responsável por:

- gerar token
- extrair subject
- validar token
- verificar expiração

#### Exemplo `JwtService.java`

```java
package com.example.projeto.security;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import javax.crypto.SecretKey;
import java.util.Date;
import java.util.function.Function;

@Service
public class JwtService {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private Long expiration;

    public String gerarToken(String username) {
        return Jwts.builder()
                .subject(username)
                .issuedAt(new Date())
                .expiration(new Date(System.currentTimeMillis() + expiration))
                .signWith(getSignKey())
                .compact();
    }

    public String extrairUsername(String token) {
        return extrairClaim(token, Claims::getSubject);
    }

    public boolean tokenValido(String token, String username) {
        String subject = extrairUsername(token);
        return subject.equals(username) && !tokenExpirado(token);
    }

    private boolean tokenExpirado(String token) {
        return extrairClaim(token, Claims::getExpiration).before(new Date());
    }

    private <T> T extrairClaim(String token, Function<Claims, T> resolver) {
        Claims claims = Jwts.parser()
                .verifyWith(getSignKey())
                .build()
                .parseSignedClaims(token)
                .getPayload();

        return resolver.apply(claims);
    }

    private SecretKey getSignKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

---

### Etapa 5 — Criar filtro JWT

A skill deve criar um filtro que:

1. leia o header `Authorization`
2. verifique prefixo `Bearer `
3. extraia o token
4. valide o usuário
5. autentique no contexto do Spring

#### Exemplo `JwtAuthenticationFilter.java`

```java
package com.example.projeto.security;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        final String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        String token = authHeader.substring(7);
        String username = jwtService.extrairUsername(token);

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);

            if (jwtService.tokenValido(token, userDetails.getUsername())) {
                UsernamePasswordAuthenticationToken authentication =
                        new UsernamePasswordAuthenticationToken(
                                userDetails,
                                null,
                                userDetails.getAuthorities()
                        );

                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

---

### Etapa 6 — Criar configuração de segurança

A skill deve liberar:

- `/auth/**`
- Swagger
- OpenAPI

E proteger o restante.

#### Exemplo `SecurityConfig.java`

```java
package com.example.projeto.security;

import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@RequiredArgsConstructor
@EnableMethodSecurity
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthenticationFilter;
    private final UserDetailsService userDetailsService;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http,
                                                   AuthenticationProvider authenticationProvider) throws Exception {
        return http
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers(
                                "/auth/**",
                                "/swagger-ui.html",
                                "/swagger-ui/**",
                                "/v3/api-docs/**"
                        ).permitAll()
                        .anyRequest().authenticated()
                )
                .authenticationProvider(authenticationProvider)
                .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
                .build();
    }

    @Bean
    public AuthenticationProvider authenticationProvider(PasswordEncoder passwordEncoder) {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder);
        return provider;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration configuration) throws Exception {
        return configuration.getAuthenticationManager();
    }
}
```

---

### Etapa 7 — Criar service de autenticação

A skill deve criar um service responsável por:

- autenticar usuário
- gerar token
- montar resposta

#### Exemplo `AutenticacaoService.java`

```java
package com.example.projeto.modules.auth.service;

import com.example.projeto.modules.auth.domain.AutenticacaoRequest;
import com.example.projeto.modules.auth.domain.AutenticacaoResponse;
import com.example.projeto.security.JwtService;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class AutenticacaoService {

    private final AuthenticationManager authenticationManager;
    private final JwtService jwtService;

    public AutenticacaoResponse login(AutenticacaoRequest request) {
        authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                        request.getEmail(),
                        request.getSenha()
                )
        );

        String token = jwtService.gerarToken(request.getEmail());

        return AutenticacaoResponse.builder()
                .token(token)
                .tipo("Bearer")
                .build();
    }
}
```

---

### Etapa 8 — Criar controller de autenticação

#### Exemplo `AutenticacaoController.java`

```java
package com.example.projeto.modules.auth.controller;

import com.example.projeto.modules.auth.domain.AutenticacaoRequest;
import com.example.projeto.modules.auth.domain.AutenticacaoResponse;
import com.example.projeto.modules.auth.service.AutenticacaoService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/auth")
@RequiredArgsConstructor
public class AutenticacaoController {

    private final AutenticacaoService autenticacaoService;

    @PostMapping("/login")
    public AutenticacaoResponse login(@RequestBody AutenticacaoRequest request) {
        return autenticacaoService.login(request);
    }
}
```

---

### Etapa 9 — Criar integração com usuário

A skill deve adaptar o projeto atual para que exista uma classe de usuário compatível com Spring Security.

Ela deve garantir:

- `email` ou `username` como identificador
- `senha` criptografada
- implementação de `UserDetails` ou `UserDetailsService`

#### Exemplo `CustomUserDetailsService.java`

```java
package com.example.projeto.security;

import com.example.projeto.modules.user.domain.UsuarioEntity;
import com.example.projeto.modules.user.repository.UsuarioRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.*;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UsuarioRepository usuarioRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UsuarioEntity usuario = usuarioRepository.findByEmail(username)
                .orElseThrow(() -> new UsernameNotFoundException("Usuário não encontrado"));

        return new User(
                usuario.getEmail(),
                usuario.getSenha(),
                List.of(new SimpleGrantedAuthority("ROLE_USER"))
        );
    }
}
```

---

### Etapa 10 — Configurar Swagger com Bearer

A skill deve configurar o OpenAPI para aceitar token JWT.

#### Exemplo `OpenApiConfig.java`

```java
package com.example.projeto.config;

import io.swagger.v3.oas.models.Components;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.security.SecurityRequirement;
import io.swagger.v3.oas.models.security.SecurityScheme;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        final String securitySchemeName = "bearerAuth";

        return new OpenAPI()
                .info(new Info()
                        .title("API com JWT")
                        .version("1.0.0")
                        .description("Documentação da API com autenticação JWT"))
                .addSecurityItem(new SecurityRequirement().addList(securitySchemeName))
                .components(new Components()
                        .addSecuritySchemes(securitySchemeName,
                                new SecurityScheme()
                                        .name(securitySchemeName)
                                        .type(SecurityScheme.Type.HTTP)
                                        .scheme("bearer")
                                        .bearerFormat("JWT")));
    }
}
```

---

## ▶️ Fluxo final esperado

1. Usuário envia email e senha para `/auth/login`
2. Spring autentica credenciais
3. Sistema gera JWT
4. Sistema retorna token
5. Front envia `Authorization: Bearer TOKEN`
6. Filtro JWT valida e autentica a requisição

---

## ✅ Resultado esperado do agent

Ao executar `@criar_auth_jwt`, o agent deve entregar:

1. dependências necessárias
2. arquivos novos ou ajustados
3. configuração de segurança
4. endpoint de login
5. geração e validação do token
6. integração com usuário
7. swagger com bearer
8. observações finais de segurança

---

## 🚫 Anti-patterns

Não fazer:

- salvar senha sem criptografia
- retornar senha no response
- gerar token sem autenticar antes
- colocar regra de autenticação no controller
- deixar todas as rotas públicas
- esquecer Swagger/OpenAPI no whitelist
- misturar autenticação JWT com sessão stateful sem intenção clara

---

## 🔒 Observações de segurança

1. A chave JWT deve ser forte e em Base64.
2. Nunca versionar segredo real no repositório.
3. A senha deve ser comparada via `PasswordEncoder`.
4. O tempo de expiração deve ser configurável.
5. Se o projeto usar cookie em vez de header, adaptar o filtro com cautela.
6. Em produção, considerar refresh token, revogação e rotação de chave.

---

## 💡 Regra de ouro

> "JWT só entra depois que a identidade foi validada."
