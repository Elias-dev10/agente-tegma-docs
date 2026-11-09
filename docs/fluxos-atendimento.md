# Fluxos de Atendimento

## Abertura de chamado

```mermaid
sequenceDiagram
    actor P as Colaborador
    participant B as Borda do Teams
    participant M as Teams (diretório)
    participant C as Cervello

    P->>B: "abre um chamado, meu notebook não liga"
    B->>B: escolhe o serviço do catálogo<br/>(a IA só escolhe entre opções da lista)
    B-->>P: até 3 perguntas para completar a descrição
    P->>B: respostas
    B->>M: e-mail de quem está pedindo
    B-->>P: resumo: categoria · tipo · assunto · solicitante<br/>"Ainda não abri nada. Posso abrir?"
    P->>B: "sim"
    B->>C: cria o chamado em nome do e-mail do Teams
    C-->>B: número do chamado
    B-->>P: ✅ Chamado N
```

| Risco | Controle |
|---|---|
| Abrir chamado em nome de outra pessoa | O solicitante é o e-mail do diretório do Teams, nunca um texto digitado |
| IA inventar categoria ou campos | A IA só escolhe um serviço de uma **lista fechada**. Todos os campos enviados ao Cervello vêm de tabelas validadas |
| Abertura sem intenção | Confirmação **escrita** depois de um resumo do que será aberto. Emoji não confirma |
| Chamado duplicado por clique repetido | Cada fluxo tem um identificador único, invalidado ao criar o chamado |
| Arquivo malicioso anexado | Só PNG/JPG, conferidos pelo conteúdo real do arquivo; até 8 MB e 3 arquivos; nome sanitizado; download só de domínios da Microsoft |

## Consulta de chamados

```mermaid
sequenceDiagram
    actor P as Colaborador
    participant B as Borda do Teams
    participant M as Teams (diretório)
    participant C as Cervello

    P->>B: "meus chamados" ou "chamado 69702"
    B->>M: e-mail de quem está pedindo
    B->>C: consulta filtrada por esse e-mail
    C-->>B: apenas chamados dessa pessoa
    B-->>P: lista ou detalhe
```

| Risco | Controle |
|---|---|
| Ver chamado de outra pessoa | O filtro é o e-mail do diretório e é aplicado **pelo próprio Cervello**. Um e-mail digitado é ignorado |
| Descobrir se um chamado existe | "Não existe" e "não é seu" recebem a **mesma resposta** |
| Injeção pelo texto de um chamado | O conteúdo dos chamados nunca é enviado à IA |

!!! success "Verificado em uso real"
    Quatro pessoas na mesma reunião do Teams pediram "meus chamados". Cada uma recebeu apenas os
    próprios chamados.

## Atendimento pela base de conhecimento

```mermaid
flowchart TD
    A[Relato do problema] --> B[Consulta à base de conhecimento]
    B --> C{"Há procedimento que<br/>a própria pessoa executa?"}
    C -- sim --> D[Mostra o passo a passo<br/>e pergunta se funcionou]
    D -- resolveu --> E([Encerra sem chamado])
    D -- "não resolveu" --> F[Abertura de chamado<br/>com o que já foi tentado]
    C -- não --> F
```

!!! danger "Procedimento do Service Desk nunca chega ao colaborador"
    Cada artigo da base declara quem o executa: `usuario`, `analista` ou `misto`. **Só artigos
    `usuario` são mostrados.** Procedimentos de analista, como desbloqueio no AD ou senha
    temporária, poderiam servir de roteiro de engenharia social. Eles vão no máximo como
    referência, dentro do chamado, para o analista. A regra é aplicada por código e coberta por
    teste.

- No máximo 2 perguntas e 2 tentativas antes de propor o chamado.
- Se a base estiver fora do ar, a conversa segue normalmente, sem a base.
