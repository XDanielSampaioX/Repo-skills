# 🧠 Skill: Aplicar arquitetura em camadas no projeto atual (@camadas)

## 🎯 Objetivo
Reorganizar o projeto atual em camadas técnicas tradicionais, separando responsabilidades por tipo de classe.

---

## 🧾 Input esperado

Aplique @camadas no projeto atual

Exemplo:
- Aplique @camadas no projeto atual
- Reorganize este projeto com @camadas

---

## 📌 Resultado esperado

O agent deve:

1. analisar a estrutura atual
2. identificar responsabilidades técnicas de cada arquivo
3. mover classes para camadas globais
4. ajustar packages e imports
5. preservar o comportamento existente

---

## 🏗️ Estrutura alvo

```text
src/main/java/com/example/projeto/
├── config/
├── controller/
├── service/
├── repository/
├── mapper/
├── domain/
├── shared/
└── ProjetoApplication.java
```

---

## 📌 Regras obrigatórias

1. Controllers devem ficar em `controller/`.
2. Services devem ficar em `service/`.
3. Repositories devem ficar em `repository/`.
4. Mappers devem ficar em `mapper/`.
5. Entidades, requests, responses e objetos de domínio devem ficar em `domain/`.
6. Configurações globais devem ficar em `config/`.
7. Compartilhados devem ficar em `shared/`.
8. Atualizar todos os packages e imports após a reorganização.
9. Não misturar regra de negócio em `controller`.
10. Não deixar acesso direto de `controller` para `repository`.

---

## 🧩 Passo a passo

### Etapa 1 — Mapear arquivos atuais
Ler a estrutura atual e classificar cada classe pela sua responsabilidade.

### Etapa 2 — Criar camadas globais
Criar as pastas:
- `controller`
- `service`
- `repository`
- `mapper`
- `domain`
- `config`
- `shared`

### Etapa 3 — Mover classes técnicas
Exemplo:
- `UsuarioController` → `controller/`
- `UsuarioService` → `service/`
- `UsuarioRepository` → `repository/`
- `UsuarioMapper` → `mapper/`
- `UsuarioEntity`, `UsuarioRequest`, `UsuarioResponse` → `domain/`

### Etapa 4 — Isolar compartilhados
Mover para `shared/` apenas o que for reutilizável em vários fluxos.

### Etapa 5 — Isolar configurações
Mover para `config/`:
- segurança
- swagger
- cors
- serialização
- paginação

### Etapa 6 — Ajustar referências
Corrigir:
- `package`
- imports
- referências cruzadas

### Etapa 7 — Revisar acoplamento
Garantir fluxo:
`controller -> service -> repository`

---

## ✅ Quando aplicar

Use `@camadas` quando:
- quer organização simples
- o time já trabalha assim
- o projeto ainda é pequeno ou médio
- quer facilidade inicial de manutenção

---

## 🚫 Anti-patterns

Não fazer:
- service gigante centralizando tudo
- controller com regra de negócio
- repository com validação de negócio
- domain excessivamente anêmico por descuido
- misturar shared com domain sem critério

---

## 🧠 Resposta esperada do agent

O agent deve entregar:

1. estrutura atual resumida
2. estrutura final em camadas
3. classes movidas por camada
4. packages atualizados
5. pontos de atenção após a migração

---

## 💡 Regra de ouro

> "Na arquitetura em camadas, cada tipo de classe tem seu lugar."
