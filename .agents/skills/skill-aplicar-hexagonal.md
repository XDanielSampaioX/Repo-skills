# 🧠 Skill: Aplicar arquitetura hexagonal no projeto atual (@hexagonal)

## 🎯 Objetivo
Reestruturar o projeto atual para arquitetura hexagonal, separando domínio, casos de uso e adaptadores, protegendo a regra de negócio da infraestrutura.

---

## 🧾 Input esperado

Aplique @hexagonal no projeto atual

Exemplo:
- Aplique @hexagonal no projeto atual
- Reestruture este projeto com @hexagonal

---

## 📌 Resultado esperado

O agent deve:

1. identificar a regra de negócio central
2. separar domínio puro de infraestrutura
3. criar ports de entrada e saída
4. mover controllers para adapters de entrada
5. mover JPA/repositories para adapters de saída
6. criar ou ajustar casos de uso
7. atualizar imports e packages

---

## 🏗️ Estrutura alvo

```text
src/main/java/com/example/projeto/
├── config/
├── shared/
├── modules/
│   └── usuario/
│       ├── domain/
│       │   ├── model/
│       │   ├── port/
│       │   │   ├── in/
│       │   │   └── out/
│       │   └── service/
│       ├── application/
│       │   └── usecase/
│       ├── adapter/
│       │   ├── in/
│       │   │   └── web/
│       │   └── out/
│       │       └── persistence/
│       └── mapper/
└── ProjetoApplication.java
```

---

## 📌 Regras obrigatórias

1. O domínio não pode depender de Spring.
2. O domínio não pode depender de JPA.
3. O domínio não pode conhecer controller nem framework web.
4. Toda entrada deve passar por `port/in`.
5. Toda saída deve passar por `port/out`.
6. Controllers devem ficar em `adapter/in`.
7. Persistência deve ficar em `adapter/out`.
8. Casos de uso devem orquestrar a regra.
9. Models de domínio e entidades JPA devem ser separados.
10. Atualizar packages, imports e mapeamentos após a migração.

---

## 🧩 Passo a passo

### Etapa 1 — Identificar o domínio puro
Localizar entidades e regras que representam o negócio central.

### Etapa 2 — Criar models de domínio
Separar classes puras sem dependência de framework.

### Etapa 3 — Criar ports
Criar interfaces para entrada e saída:
- `port/in`
- `port/out`

### Exemplo
- `CriarUsuarioUseCase`
- `BuscarUsuarioUseCase`
- `UsuarioRepositoryPort`

### Etapa 4 — Criar casos de uso
Criar implementações em `application/usecase/` que usam as ports.

### Etapa 5 — Criar adapters de entrada
Mover controllers para:
```text
adapter/in/web
```

### Etapa 6 — Criar adapters de saída
Mover persistência para:
```text
adapter/out/persistence
```

### Etapa 7 — Separar JPA do domínio
Criar entidades JPA próprias quando necessário.

### Etapa 8 — Criar mappers
Mapear:
- request ↔ domain model
- domain model ↔ response
- domain model ↔ jpa entity

### Etapa 9 — Ajustar injeções e implementações
Garantir que os adapters implementem as ports corretamente.

---

## ✅ Quando aplicar

Use `@hexagonal` quando:
- o projeto já cresceu
- o domínio tem muita regra de negócio
- quer testes melhores
- quer baixo acoplamento com Spring/JPA
- quer separar fortemente negócio e infraestrutura

---

## ⚠️ Atenção

Essa migração é mais profunda que modular e camadas.  
Ela pode exigir criação de novas interfaces, modelos e mappers.

---

## 🚫 Anti-patterns

Não fazer:
- manter `@Entity` como model de domínio
- chamar JPA direto do controller
- usar ports só por nome sem isolamento real
- deixar caso de uso dependente de framework
- misturar adapter com domain

---

## 🧠 Resposta esperada do agent

O agent deve entregar:

1. diagnóstico da estrutura atual
2. proposta de separação por domínio, application e adapters
3. lista de classes que viram ports
4. lista de classes que viram adapters
5. novos mappers necessários
6. observações sobre o impacto da migração

---

## 💡 Regra de ouro

> "Na arquitetura hexagonal, o domínio continua de pé mesmo se o framework cair."
