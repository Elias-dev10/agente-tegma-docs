# Superfície de Ataque

## O que está exposto

Há um único serviço público, a borda do Teams. O IAM do Google Cloud vale para o serviço
inteiro, não para cada rota, e o Teams precisa alcançar o serviço sem credencial do Google. Por
isso **cada rota tem autenticação própria na aplicação**.

| Rota | Função | Sem credencial | Credencial exigida |
|---|---|---|---|
| `POST /api/messages` | Mensagens do Teams | **401** | Token assinado pela Microsoft |
| `POST /interno/processa` | Entrega da fila interna | **401** | Token do Google da conta da fila |
| `POST /api/notify` | Aviso proativo por sistema autorizado | **401** | Chave de integração |
| `GET/POST /api/alias` | Cadastro de destinatários de avisos | **401** | Chave de integração |
| `POST /chat` | Testes de integração | **401** | Chave de integração |
| `GET /health` | Status de saúde | 200 `{"status": "ok"}` | Detalhes só com a chave |
| `/docs`, `/openapi.json` | Documentação da API | **404** | Desligada em produção |

!!! success "A superfície é verificada por teste"
    Um teste automatizado lista todas as rotas da aplicação e exige 401 sem credencial. Se surgir
    uma rota nova sem autenticação, o build quebra e a versão não vai para produção.

## O que está fechado

| Componente | Acesso de fora |
|---|---|
| Agente-Tegma | **Bloqueado pelo Google (403).** Só a conta de serviço da borda pode chamá-lo |
| Base de procedimentos | **Bloqueada pelo Google (403).** Só a conta do agente pode chamá-la |
| Redis | Inacessível. Só a rede interna, com senha |
| Máquinas virtuais | Sem SSH ou RDP público. Acesso apenas via Identity-Aware Proxy |
| Bancos de auditoria (BigQuery) | Acesso apenas por IAM nominal |

## O que um avaliador **não** vai encontrar

- Credenciais no código, em variáveis em texto ou em log.
- Documentação da API, mapa de rotas ou versão de software expostos.
- Mensagens de erro diferentes conforme o motivo da recusa: todas as recusas de autenticação são idênticas.
- Erro 500 em falhas de autenticação: toda falha vira 401.
- Rota interna acessível sem token do Google.
