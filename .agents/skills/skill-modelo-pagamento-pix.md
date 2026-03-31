# 🧠 Skill: Integrar pagamento PIX via Mercado Pago (@modelo_pagamento_pix)

## 🎯 Objetivo
Integrar pagamento PIX em uma aplicação Java Spring + frontend web, usando Mercado Pago com geração do QR Code/copia e cola no backend e confirmação assíncrona por webhook.

---

## 🧾 Input esperado

Implemente pagamento PIX @modelo_pagamento_pix

---

## 📌 Estratégia recomendada

Usar Checkout Transparente com geração do pagamento PIX no backend e exibição do QR Code/código copia e cola no frontend.

---

## 📦 Resultado esperado

1. Configuração de credenciais
2. Endpoint para criar pagamento PIX
3. Retorno com QR Code e/ou código copia e cola
4. Persistência do pedido local
5. Webhook de confirmação
6. Atualização automática de status

---

## 🧩 Fluxo

### Etapa 1 — Criar credenciais
Configurar:
- Public Key no frontend se usar Brick
- Access Token no backend

---

### Etapa 2 — Criar pedido local
Salvar:
- id do pedido
- external_reference
- valor
- status inicial `AGUARDANDO_PAGAMENTO`

---

### Etapa 3 — Criar pagamento PIX no backend
Chamar o Mercado Pago com:
- transaction_amount
- description
- payment_method_id = pix
- payer.email
- first_name
- last_name
- identification (quando exigido)
- external_reference
- notification_url

---

### Etapa 4 — Retornar dados do PIX
O backend deve devolver para o frontend:
- payment_id
- status
- qr_code
- qr_code_base64
- external_reference

---

### Etapa 5 — Exibir QR Code
O frontend deve:
- renderizar QR Code
- mostrar opção de copiar o código PIX
- informar status aguardando pagamento

---

### Etapa 6 — Receber webhook
Criar endpoint para receber notificações do Mercado Pago.

---

### Etapa 7 — Confirmar via consulta
Ao receber notificação:
1. validar assinatura
2. buscar o pagamento no Mercado Pago
3. atualizar status local do pedido

---

## 📌 Estrutura sugerida

```text
/modules/payment/
  /controller
    PixController.java
    WebhookPagamentoController.java
  /service
    PagamentoPixService.java
    WebhookPagamentoService.java
  /domain
    PagamentoPixRequest.java
    PagamentoPixResponse.java
  /repository
    PagamentoRepository.java
```

---

## 🔐 Regras obrigatórias

1. O pedido local deve nascer antes da confirmação do PIX.
2. O frontend não deve confirmar pagamento sozinho.
3. A confirmação oficial deve vir por webhook + consulta.
4. Sempre usar `external_reference`.
5. O QR Code/copia e cola deve vir do backend.
6. Tratar expiração e abandono do pagamento.

---

## ⚠️ Estados comuns

- pending
- approved
- cancelled
- expired

---

## 🚫 Anti-patterns

- marcar como pago sem webhook
- não persistir payment_id
- depender apenas de polling do frontend
- ignorar expiração do PIX
- não validar autenticidade do webhook

---

## 💡 Regra de ouro

> "PIX nasce no backend, aparece no frontend e confirma por webhook."
