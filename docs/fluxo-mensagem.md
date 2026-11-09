# Fluxo de uma Mensagem

Toda mensagem passa pelos mesmos controles, na mesma ordem, antes de qualquer processamento.

```mermaid
sequenceDiagram
    participant T as Teams (Microsoft)
    participant B as Borda do Teams
    participant R as Redis
    participant Q as Fila de processamento
    participant P as Processamento

    T->>B: mensagem + token assinado pela Microsoft
    Note over B: 1. limite de requisições por origem<br/>(antes da autenticação)
    B->>B: 2. valida o token (assinatura, emissor, destino, validade)
    B->>B: 3. confere se a origem declarada bate com o token
    B->>R: 4. descarta mensagem repetida
    Note over B: 5. limite por pessoa e proteção contra sobrecarga
    B->>Q: 6. enfileira
    B-->>T: recebido (em milissegundos)
    Q->>P: 7. entrega autenticada (token do Google)
    P->>P: 8. decide o caminho e executa
    P->>T: 9. resposta ao colaborador
```

## Etapas e controles

| # | Etapa | Controle de segurança | Se falhar |
|---|---|---|---|
| 1 | Limite por origem | Teto de requisições por IP e global, contado **antes** da autenticação | Recusa com 429 |
| 2 | Autenticação | Token RS256 assinado pela Microsoft, com emissor, destino e validade conferidos | **401**, com a mesma resposta para qualquer motivo |
| 3 | Origem | O endereço de resposta declarado tem de bater com o do token e pertencer à Microsoft (anti-SSRF) | **401** |
| 4 | Duplicidade | Cada mensagem é processada uma única vez | Ignora a repetição |
| 5 | Abuso | 15 mensagens por minuto por pessoa e teto de fila | Aviso à pessoa |
| 6–7 | Fila | A entrega interna exige token do Google emitido para a conta da fila | **401** |
| 8 | Decisão | Pedidos óbvios ("abrir chamado", "meus chamados", "cancelar") são resolvidos **sem IA**. O resto vai ao Agente-Tegma | — |
| 9 | Resposta | Enviada só a endereços da Microsoft. Em grupo, só responde quando o assistente é @mencionado | — |

!!! note "Responder rápido não reduz a segurança"
    O Teams exige confirmação em poucos segundos. Por isso a borda confirma o recebimento e
    entrega o processamento a uma fila. A rota interna que recebe da fila é tão protegida quanto a
    pública: ela só aceita chamadas assinadas pelo Google para a conta da fila.
