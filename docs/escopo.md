# Escopo do Sistema

## Quem usa

| Perfil | Interação |
|---|---|
| **Colaborador Tegma** | Conversa com o Agente-Tegma no Teams, em chat individual, grupo ou reunião. Em grupo, o assistente só responde quando é @mencionado |
| **Analista do Service Desk** | Recebe no Cervello o chamado já classificado e descrito |
| **Administrador** (lista fechada de e-mails) | Envia avisos proativos a pessoas ou grupos |

## O que o sistema faz

| Funcionalidade | Como funciona |
|---|---|
| **Responder dúvidas de TI** | Consulta a base de conhecimento e mostra um passo a passo **apenas se a própria pessoa puder executá-lo** |
| **Abrir chamado** | A pessoa descreve o problema. O sistema escolhe o serviço do catálogo, faz até 3 perguntas, mostra o resumo e só abre depois de um "sim" escrito |
| **Anexar prints** | Até 3 imagens PNG/JPG de até 8 MB, validadas antes do envio ao Cervello |
| **Consultar chamados** | Lista ou detalha **apenas os chamados da própria pessoa** |
| **Avisos proativos** | Somente administradores, com resumo e confirmação |

## O que o sistema **não** faz

!!! warning "Limites deliberados"
    - **Não executa ações em sistemas da Tegma.** Não reseta senha, não libera acesso e não altera cadastro. Esses pedidos viram chamado.
    - **Não mostra procedimentos internos do Service Desk** (por exemplo, desbloqueio no AD ou senha temporária).
    - **Não pede nem armazena senha, token, MFA, PIN ou cartão.**
    - **Não consulta chamados de outra pessoa**, mesmo que alguém digite outro e-mail.
    - **Não aceita confirmação por emoji.** Abrir chamado exige resposta escrita ou clique no botão de confirmação.
    - **Não sai do escopo de suporte de TI.** Pedidos de código, textos, traduções ou pesquisas gerais são recusados.
