# Guia para Avaliadores

!!! success "Testes autorizados"
    A equipe de segurança da Tegma pode avaliar a solução, desde que combine **janela e escopo**
    com a iK. Isso evita que o tráfego de teste dispare alertas como incidente ou gere chamados
    reais no Cervello.

## O que esperar de cada teste

| Teste | Resultado esperado |
|---|---|
| **Varredura de portas e serviços** | Apenas HTTPS (443) no serviço público. Serviços internos respondem 403 do Google |
| **Rotas sem credencial** | 401 em todas, exceto `/health`, que devolve só `{"status": "ok"}`. Documentação da API: 404 |
| **Token forjado, expirado ou de outro emissor** | 401, sempre com a mesma mensagem, qualquer que seja o motivo. Nunca 500 |
| **Endereço de resposta adulterado** (SSRF) | 401 |
| **Chamada direta à rota interna** | 401 sem token do Google da conta da fila |
| **Rajada de requisições** | 429 após o teto por IP. O alerta de sondagem dispara |
| **Scan de aplicação web** (ZAP, Burp, nikto) | Cabeçalhos de segurança presentes. Sem páginas, formulários ou documentação expostos |
| **Pedir chamados de outra pessoa** ("meus chamados do fulano@…") | Devolve os chamados **de quem pediu**: o e-mail digitado é ignorado |
| **Consultar número de chamado alheio** | "Não encontrei esse chamado no seu nome", a mesma resposta dada para chamado inexistente |
| **Prompt injection e jailbreak** | O agente recusa ou responde dentro do escopo. Nenhuma ação é executada, porque a IA não tem ferramenta de escrita |
| **Pedir procedimento do Service Desk** ("como desbloquear usuário no AD") | O agente diz que é com a equipe de TI e oferece chamado |
| **Pedir aviso proativo sem ser administrador** | Recusa e registro de auditoria |
| **Anexar arquivo não permitido** (.exe, .svg, .html, imagem falsa) | Recusado com motivo. Nada é enviado ao Cervello |
| **Confirmar abertura de chamado com emoji** | Não abre. Exige resposta escrita ou botão |

## Ferramentas de apoio

| Ferramenta | Uso |
|---|---|
| **Verificador de postura** | Script somente leitura que confere IAM, firewall, superfície exposta, cabeçalhos e recusas no ambiente real. A iK pode executá-lo junto com a equipe da Tegma |
| **Inventário de rotas** | A tabela em [Superfície de Ataque](superficie-ataque.md) é a lista completa. Recomendamos usá-la como escopo do teste |
| **Rastreio por identificador** | Cada requisição de teste pode ser localizada nos logs pelo identificador de rastreio, para correlacionar com o resultado observado |

## Pontos para a avaliação da Tegma

| Ponto | Situação |
|---|---|
| Ambiente do Cervello | A integração usa a homologação. A troca para produção inclui repetir os testes de isolamento por usuário |
| Identidade do aplicativo no Azure | A migração para o tenant da Tegma troca só as credenciais do aplicativo. O endereço e os controles não mudam |
| Privacidade | Base legal, aviso aos usuários e processamento de texto pela OpenAI (ver [Dados, Privacidade e Auditoria](dados-privacidade.md)) |
| Redis | Instância única em rede privada. Se cair, o atendimento continua, mas sem memória da conversa |
| Proteção de borda gerenciada (WAF) | Avaliada e não adotada: o limite por IP real na aplicação já absorveu picos de varredura sem indisponibilidade. Pode ser reavaliada pela Tegma |
