# 🧠 Skill: Aplicar arquitetura modular no projeto atual (@modular)

## 🎯 Objetivo
Reorganizar o projeto atual para arquitetura modular, preservando o domínio existente e redistribuindo os arquivos por feature.

---

## 🧾 Input esperado

Aplique @modular no projeto atual

Exemplo:
- Aplique @modular no projeto atual
- Reorganize este projeto com @modular

---

## 📌 Resultado esperado

O agent deve:

1. analisar a estrutura atual do projeto
2. identificar features e domínios existentes
3. mover arquivos para módulos por feature
4. manter configurações globais fora dos módulos
5. ajustar imports, packages e referências
6. entregar a nova árvore de pastas e os arquivos impactados

---

## 🏗️ Estrutura alvo

```text
src/main/java/com/example/projeto/
├── config/
├── shared/
├── modules/
│   ├── usuario/
│   │   ├── controller/
│   │   ├── service/
│   │   ├── repository/
│   │   ├── mapper/
│   │   └── domain/
│   ├── pedido/
│   │   ├── controller/
│   │   ├── service/
│   │   ├── repository/
│   │   ├── mapper/
│   │   └── domain/
│   └── produto/
│       ├── controller/
│       ├── service/
│       ├── repository/
│       ├── mapper/
│       └── domain/
└── ProjetoApplication.java
```

---

## 📌 Regras obrigatórias

1. A organização principal deve ser por feature.
2. Cada módulo deve conter:
   - `controller`
   - `service`
   - `repository`
   - `mapper`
   - `domain`
3. Configurações globais devem ir para `config/`.
4. Classes compartilhadas devem ir para `shared/`.
5. Não deixar `controller`, `service` e `repository` globais.
6. Não alterar regra de negócio sem necessidade.
7. Atualizar `package` e imports de todos os arquivos movidos.
8. Preservar nomes dos arquivos existentes sempre que possível.

---

## 🧩 Passo a passo

### Etapa 1 — Mapear domínios atuais
Identificar entidades, controllers, services, repositories e mappers existentes.

### Etapa 2 — Agrupar por feature
Separar os arquivos por domínio principal.

### Exemplo
- `UsuarioController`, `UsuarioService`, `UsuarioRepository`, `UsuarioMapper`, `UsuarioEntity`
  → módulo `usuario`
- `PedidoController`, `PedidoService`, `PedidoRepository`
  → módulo `pedido`

### Etapa 3 — Criar `modules/`
Criar a pasta raiz `modules/` e, dentro dela, um módulo para cada feature.

### Etapa 4 — Mover arquivos
Mover cada classe para a feature correta.

### Etapa 5 — Isolar compartilhados
Mover para `shared/` apenas o que for realmente comum:
- exceptions
- responses genéricas
- enums globais
- utilitários
- abstrações compartilhadas

### Etapa 6 — Isolar configurações
Mover para `config/`:
- Swagger
- Security
- CORS
- Pageable
- Jackson
- OpenAPI

### Etapa 7 — Ajustar imports e packages
Atualizar todos os packages afetados após a reorganização.

### Etapa 8 — Validar dependências entre módulos
Evitar acoplamento desnecessário entre features.

---

## ✅ Quando aplicar

Use `@modular` quando:
- o projeto já existe e está bagunçado por camadas globais
- quer evolução por feature
- quer manter Spring tradicional sem complexidade excessiva
- quer organização limpa e escalável

---

## 🚫 Anti-patterns

Não fazer:
- misturar arquivos de features diferentes no mesmo módulo
- criar `service` global e `modules` ao mesmo tempo
- jogar tudo em `shared/`
- mover regra de negócio para controller só por reorganização
- quebrar imports sem corrigir referência

---

## 🧠 Resposta esperada do agent

O agent deve entregar:

1. árvore atual resumida
2. árvore alvo
3. lista de arquivos movidos
4. pacotes atualizados
5. observações sobre arquivos que devem ir para `shared` ou `config`

---

## 💡 Regra de ouro

> "Na arquitetura modular, a feature vem primeiro e a camada vem depois."
