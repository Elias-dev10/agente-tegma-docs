# Agente-Tegma — Documentação de Segurança

Site MkDocs (Material + Mermaid) com o escopo, os fluxos e as camadas de segurança do atendimento
de TI da Tegma no Microsoft Teams, para avaliação pela equipe de segurança da Tegma.

> Não contém credenciais, chaves nem valores de segredo.

## Pré-visualizar localmente

```bash
python -m venv .venv && . .venv/Scripts/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
mkdocs serve        # http://127.0.0.1:8000
```

## Publicar no GitHub Pages

```bash
mkdocs gh-deploy --force      # https://elias-dev10.github.io/agente-tegma-docs/
```
