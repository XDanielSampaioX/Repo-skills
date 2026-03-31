# 🧠 Skill: Integrar Mercado Pago completo (@integrar_mercado_pago)

## 🎯 Objetivo
Integrar Mercado Pago completo em uma aplicação Java Spring + frontend web, cobrindo pagamento com cartão, pagamento via PIX, persistência local do pedido, webhook de confirmação e atualização segura do status.

---

## 🧾 Input esperado

Implemente Mercado Pago completo @integrar_mercado_pago

Exemplos:
- Integre Mercado Pago @integrar_mercado_pago
- Adicione pagamento com cartão e PIX @integrar_mercado_pago
- Implemente checkout completo com Mercado Pago @integrar_mercado_pago

---

## 📦 Resultado esperado

A skill deve gerar ou ajustar:

1. configuração de credenciais do Mercado Pago
2. módulo de pagamento
3. fluxo de pagamento com cartão
4. fluxo de pagamento com PIX
5. criação de pedido local
6. persistência do pagamento
7. endpoint de webhook
8. validação da notificação
9. consulta oficial do pagamento antes da confirmação
10. atualização de status do pedido
11. estrutura pronta para expansão

---

## 📌 Stack assumida

- Java
- Spring Boot
- Lombok
- MapStruct
- Swagger/OpenAPI
- Banco relacional
- Frontend web (Next.js, React ou similar)

---

## 🏗️ Estrutura sugerida

```text
/modules/payment/
  /controller
    PagamentoController.java
    PixController.java
    WebhookPagamentoController.java
  /service
    PagamentoCartaoService.java
    PagamentoPixService.java
    WebhookPagamentoService.java
    MercadoPagoClientService.java
  /domain
    PagamentoCartaoRequest.java
    PagamentoPixRequest.java
    PagamentoResponse.java
    WebhookMercadoPagoRequest.java
    PagamentoEntity.java
  /repository
    PagamentoRepository.java
  /mapper
    PagamentoMapper.java

/modules/order/
  /controller
  /service
  /repository
  /domain
    PedidoEntity.java
    PedidoResponse.java
```

---

## 📌 Regras obrigatórias

1. O pedido local deve existir antes da confirmação final do pagamento.
2. Todo pagamento deve possuir `external_reference`.
3. O frontend nunca deve definir sozinho que o pedido foi pago.
4. O backend deve confirmar o pagamento por webhook + consulta oficial.
5. O Access Token do Mercado Pago deve ficar apenas no backend.
6. A Public Key deve ser usada apenas no frontend.
7. O backend deve persistir:
   - payment_id
   - external_reference
   - status
   - status_detail
   - valor
   - método de pagamento
   - usuário
   - pedido vinculado
8. O backend deve registrar estados intermediários com segurança.
9. O sistema deve suportar cartão e PIX no mesmo módulo.
10. O webhook nunca deve aprovar pedido sem reconsulta ao Mercado Pago.

---

# 🔁 Fluxo completo da integração

## Fluxo geral

1. usuário cria ou inicia um pedido
2. backend salva pedido local
3. backend gera `external_reference`
4. frontend escolhe forma de pagamento
5. backend ou frontend prepara os dados conforme o método
6. Mercado Pago processa o pagamento
7. backend salva retorno inicial
8. webhook notifica atualização
9. backend consulta pagamento oficial
10. pedido local é atualizado com status final

---

# 💳 Fluxo de cartão

## Estratégia
Usar tokenização do cartão no frontend.

## Passos

### Etapa 1 — Frontend renderiza componente de cartão
O frontend deve usar o SDK/Brick do Mercado Pago para capturar os dados sensíveis do cartão.

### Etapa 2 — Frontend tokeniza cartão
O frontend deve enviar ao backend apenas:
- token
- transaction_amount
- installments
- payment_method_id
- issuer_id (quando aplicável)
- payer.email
- identificação do comprador
- external_reference

### Etapa 3 — Backend cria pagamento
O backend deve chamar o Mercado Pago com os dados tokenizados.

### Etapa 4 — Backend salva retorno inicial
Salvar:
- payment_id
- status inicial
- status_detail
- external_reference
- tipo `CARTAO`

### Etapa 5 — Webhook confirma status final
O webhook deve ser tratado da mesma forma dos outros meios de pagamento.

---

# ⚡ Fluxo de PIX

## Estratégia
Gerar o pagamento PIX no backend e devolver QR Code / copia e cola ao frontend.

## Passos

### Etapa 1 — Backend cria pagamento PIX
Enviar:
- transaction_amount
- description
- payment_method_id = pix
- payer.email
- dados do comprador
- external_reference
- notification_url

### Etapa 2 — Backend salva retorno inicial
Salvar:
- payment_id
- status inicial
- qr_code
- qr_code_base64
- external_reference
- tipo `PIX`

### Etapa 3 — Frontend exibe QR Code
Frontend deve:
- renderizar QR Code
- exibir código copia e cola
- informar status aguardando pagamento

### Etapa 4 — Webhook confirma pagamento
Após a notificação, o backend consulta o pagamento e atualiza o pedido.

---

# 🧩 Modelagem sugerida

## PedidoEntity
A entidade de pedido deve possuir no mínimo:

```java
private Long id;
private String externalReference;
private BigDecimal valorTotal;
private PedidoStatus status;
private Long usuarioId;
```

## PagamentoEntity
A entidade de pagamento deve possuir no mínimo:

```java
private Long id;
private String paymentIdExterno;
private String externalReference;
private String metodoPagamento;
private BigDecimal valor;
private String status;
private String statusDetail;
private String qrCode;
private String qrCodeBase64;
private Long pedidoId;
private LocalDateTime dataCriacao;
private LocalDateTime dataAtualizacao;
```

---

# 🧩 DTOs sugeridos

## PagamentoCartaoRequest

```java
private String token;
private BigDecimal transactionAmount;
private Integer installments;
private String paymentMethodId;
private String issuerId;
private String emailComprador;
private String externalReference;
```

## PagamentoPixRequest

```java
private BigDecimal transactionAmount;
private String descricao;
private String emailComprador;
private String externalReference;
```

## PagamentoResponse

```java
private String paymentId;
private String status;
private String statusDetail;
private String externalReference;
private String qrCode;
private String qrCodeBase64;
```

---

# 🧩 Serviços esperados

## MercadoPagoClientService
Responsável por:
- centralizar chamadas ao Mercado Pago
- criar pagamento cartão
- criar pagamento PIX
- consultar pagamento por id
- encapsular headers e autenticação

## PagamentoCartaoService
Responsável por:
- validar request
- vincular pedido local
- chamar client do Mercado Pago
- persistir pagamento
- devolver resposta

## PagamentoPixService
Responsável por:
- validar request
- chamar client do Mercado Pago
- persistir dados do PIX
- devolver QR Code e status

## WebhookPagamentoService
Responsável por:
- validar notificação
- extrair id do pagamento
- consultar pagamento oficial
- atualizar pagamento local
- atualizar pedido local

---

# 🌐 Controllers esperados

## PagamentoController
Rotas sugeridas:
- `POST /pagamentos/cartao`

## PixController
Rotas sugeridas:
- `POST /pagamentos/pix`

## WebhookPagamentoController
Rotas sugeridas:
- `POST /webhooks/mercado-pago`

---

# 🔐 Segurança obrigatória

1. Nunca receber número do cartão puro no backend.
2. Nunca expor Access Token no frontend.
3. Nunca confiar apenas no retorno síncrono do pagamento.
4. Sempre consultar o pagamento oficial após o webhook.
5. Validar autenticidade da notificação recebida.
6. Usar `external_reference` para correlação.
7. Controlar idempotência para evitar reprocessamento.
8. Registrar logs de integração sem vazar dados sensíveis.

---

# 📌 Estados que devem ser tratados

## Pagamento
- pending
- approved
- in_process
- rejected
- cancelled
- refunded
- charged_back
- expired

## Pedido local
Sugestão de status:
- AGUARDANDO_PAGAMENTO
- PAGAMENTO_EM_PROCESSAMENTO
- PAGO
- RECUSADO
- CANCELADO
- EXPIRADO
- REEMBOLSADO

---

# 🔁 Regras de atualização de status

## Exemplo de mapeamento

- `approved` → pedido `PAGO`
- `pending` → pedido `AGUARDANDO_PAGAMENTO`
- `in_process` → pedido `PAGAMENTO_EM_PROCESSAMENTO`
- `rejected` → pedido `RECUSADO`
- `cancelled` → pedido `CANCELADO`
- `expired` → pedido `EXPIRADO`
- `refunded` → pedido `REEMBOLSADO`

---

# 🧩 Passo a passo de execução

## Etapa 1 — Configurar credenciais
Criar variáveis:
- `MERCADO_PAGO_ACCESS_TOKEN`
- `MERCADO_PAGO_PUBLIC_KEY`
- `MERCADO_PAGO_WEBHOOK_SECRET` (se aplicável)

## Etapa 2 — Criar módulo payment
Criar pastas e classes básicas de domínio, service, controller e repository.

## Etapa 3 — Criar pedido local
Garantir que o pedido exista antes do pagamento.

## Etapa 4 — Implementar cartão
Criar rota de backend para finalizar pagamento de cartão com token vindo do frontend.

## Etapa 5 — Implementar PIX
Criar rota para gerar pagamento PIX e devolver QR Code.

## Etapa 6 — Persistir integração
Salvar sempre:
- request relevante
- response relevante
- status
- payment_id
- external_reference

## Etapa 7 — Implementar webhook
Criar endpoint público de notificação.

## Etapa 8 — Reconsultar pagamento
Ao receber webhook, buscar o pagamento oficial antes de atualizar o sistema.

## Etapa 9 — Atualizar pedido local
Sincronizar o status do pedido com o resultado oficial.

## Etapa 10 — Expor resposta para frontend
Permitir consulta do status do pedido/pagamento quando necessário.

---

# ✅ Resposta esperada do agent

Ao executar `@integrar_mercado_pago`, o agent deve entregar:

1. estrutura do módulo de pagamento
2. modelagem de pedido e pagamento
3. fluxo de cartão
4. fluxo de PIX
5. webhook
6. regras de persistência
7. regras de atualização de status
8. observações de segurança
9. arquivos novos ou alterados
10. pontos de integração com frontend

---

# 🚫 Anti-patterns

Não fazer:

- marcar pedido como pago só porque o frontend disse
- não usar external_reference
- não salvar payment_id
- não consultar o pagamento após webhook
- usar Access Token no frontend
- capturar cartão puro no backend
- ignorar expiração do PIX
- não tratar reprocessamento de webhook
- misturar pedido e pagamento sem separação

---

# 💡 Regra de ouro

> "Pedido é interno, pagamento é externo, confirmação é oficial."
