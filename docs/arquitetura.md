# Arquitetura e Topologia

## Componentes

| Componente | Papel | Exposição |
|---|---|---|
| **Borda do Teams** (`tegma-bot-service`) | Recebe as mensagens, valida assinatura e identidade, decide o caminho e executa as ações (abrir e consultar chamado) | **Pública**, porque o Teams precisa alcançá-la. Toda rota exige credencial |
| **Agente-Tegma** | Redige a conversa e consulta a base de procedimentos | Privado: só a borda pode chamá-lo |
| **Base de procedimentos** | Busca nos procedimentos internos da Tegma | Privada: só o Agente-Tegma pode chamá-la |
| **Fila de processamento** (Cloud Tasks) | Entrega cada mensagem para processamento, com novas tentativas | Interna do Google Cloud |
| **Redis** | Estado da conversa, controle de duplicidade e limites de uso | Rede privada (VPC), com senha |
| **BigQuery** | Telemetria e auditoria | Acesso por IAM nominal |
| **Secret Manager** | Cofre de credenciais | Acesso nominal por serviço |

## Diagrama

```mermaid
flowchart TB
    subgraph EXT[" Externo "]
        USER["Colaborador<br/>Microsoft Teams"]
        MS["Microsoft<br/>assina cada mensagem"]
        CERV["ITSM Cervello"]
        KB["API Base de Conhecimento"]
        OAI["OpenAI<br/>modelo de linguagem"]
    end

    subgraph GCP[" Google Cloud — tegma-teams · São Paulo "]
        subgraph PUB[" Público "]
            BOT["Borda do Teams<br/>valida assinatura e identidade"]
        end
        Q["Fila de processamento"]
        subgraph PRIV[" Privado — acesso só por IAM "]
            AG["Agente-Tegma"]
            BV["Base de procedimentos"]
        end
        REDIS[("Redis<br/>rede privada")]
        SM["Secret Manager"]
        LOG[("BigQuery<br/>auditoria")]
    end

    USER --> MS -->|"mensagem assinada"| BOT
    BOT -->|enfileira| Q -->|"entrega autenticada"| BOT
    BOT --> AG --> BV
    BOT <--> REDIS
    BOT --> CERV
    BOT --> KB
    BOT --> OAI
    AG --> OAI
    BOT -->|resposta| MS --> USER
    SM -. credenciais .-> BOT
    SM -. credenciais .-> AG
    BOT -.-> LOG
```

!!! note "Princípio central"
    A IA **conversa**, mas não **age**. Abrir chamado, consultar chamado e enviar avisos são
    executados por código determinístico na borda, com a identidade vinda do Teams e com
    confirmação escrita. O Agente-Tegma só tem acesso de leitura à base de procedimentos.

## Rede e acesso

```mermaid
flowchart LR
    subgraph INTERNET["Internet"]
        TEAMS["Microsoft Teams"]
        ANY["Qualquer origem"]
    end
    subgraph RUN["Google Cloud"]
        BOT["Borda do Teams<br/>credencial exigida na aplicação"]
        AG["Agente-Tegma<br/>chamável só pela borda"]
        BV["Base de procedimentos<br/>chamável só pelo agente"]
        R["Redis<br/>só rede interna"]
    end
    TEAMS -->|"assinatura válida → atende"| BOT
    ANY -->|"sem credencial → 401"| BOT
    ANY -.->|"bloqueado pelo IAM → 403"| AG
    BOT --> AG --> BV
    BOT --> R
```

| Controle | Efeito |
|---|---|
| Uma conta de serviço por componente | Cada serviço só alcança o que precisa. Nenhuma conta tem papel amplo de projeto |
| Serviços internos privados | Requisição sem identidade autorizada é recusada pelo Google (403) antes de chegar ao código |
| Redis sem exposição | Porta acessível só pela rede interna, com senha obrigatória |
| Acesso administrativo | SSH e RDP padrão desabilitados. Acesso a máquina só via Identity-Aware Proxy (autenticação Google) |
