# 🧠 Skill: Integrar pagamento com cartão via Mercado Pago (@modelo_pagamento_cartao)

## 🎯 Objetivo
Integrar pagamento por cartão em uma aplicação Java Spring + frontend web, usando Mercado Pago com captura segura dos dados do cartão no frontend e criação do pagamento no backend.

---

## 🧾 Input esperado

Implemente pagamento com cartão @modelo_pagamento_cartao

---

## 📌 Estratégia recomendada

Usar Checkout Transparente com Card Payment Brick no frontend e processamento do pagamento no backend.

---

## 📦 Resultado esperado

1. Configuração de credenciais do Mercado Pago
2. Frontend com Brick de cartão
3. Tokenização do cartão no frontend
4. Backend recebendo apenas os dados necessários
5. Chamada ao Mercado Pago para criar pagamento
6. Persistência do pedido local
7. Webhook para confirmação do status
8. Atualização do pedido conforme status do pagamento

---

## 🧩 Fluxo

### Etapa 1 — Criar credenciais
Configurar:
- Public Key no frontend
- Access Token no backend

---

### Etapa 2 — Frontend com MercadoPago.js
Inicializar o SDK e renderizar o Card Payment Brick.

O frontend deve enviar ao backend apenas:
- token
- transaction_amount
- installments
- payment_method_id
- payer.email
- issuer_id (quando aplicável)
- identificação do comprador
- referência do pedido

---

### Etapa 3 — Backend cria pagamento
O backend deve chamar o endpoint de pagamento do Mercado Pago ou SDK server-side usando os dados recebidos do frontend.

Campos mínimos obrigatórios:
- token
- transaction_amount
- installments
- payment_method_id
- payer.email

---

### Etapa 4 — Criar pedido local
Antes ou junto da tentativa de pagamento, salvar no banco:
- id do pedido
- external_reference
- valor
- status local inicial
- id do usuário
- itens

---

### Etapa 5 — Persistir retorno do pagamento
Salvar:
- payment_id do Mercado Pago
- status
- status_detail
- data de criação
- external_reference

---

### Etapa 6 — Configurar Webhook
Criar endpoint para receber notificações de pagamento e atualizar o pedido local.

---

### Etapa 7 — Confirmar pagamento por consulta
Ao receber o webhook:
1. validar a assinatura
2. consultar o pagamento pelo id informado
3. atualizar o pedido local com base no status oficial

---

## 📌 Estrutura sugerida

```text
/modules/payment/
  /controller
    PagamentoController.java
    WebhookPagamentoController.java
  /service
    PagamentoService.java
    MercadoPagoCartaoService.java
    WebhookPagamentoService.java
  /domain
    PagamentoCartaoRequest.java
    PagamentoResponse.java
    WebhookMercadoPagoRequest.java
  /mapper
  /repository
    PagamentoRepository.java
```

---

## 🔐 Regras obrigatórias

1. Nunca capturar número do cartão diretamente no backend.
2. O frontend deve tokenizar o cartão com Mercado Pago.
3. O backend nunca deve confiar apenas no retorno síncrono.
4. O status final do pedido deve ser confirmado por webhook + consulta do pagamento.
5. Sempre usar `external_reference` para vincular pagamento ao pedido local.
6. Nunca aprovar pedido apenas porque o frontend disse que pagou.
7. Salvar logs mínimos de integração e erros.

---

## ⚠️ Estados comuns do pagamento

- pending
- approved
- in_process
- rejected
- cancelled
- refunded
- charged_back

---

## 🚫 Anti-patterns

- receber número do cartão puro no backend
- marcar pedido como pago sem webhook/consulta
- não salvar external_reference
- não validar assinatura do webhook
- usar access token no frontend

---

## 💡 Regra de ouro

> "Cartão deve ser tokenizado no frontend e confirmado no backend."
