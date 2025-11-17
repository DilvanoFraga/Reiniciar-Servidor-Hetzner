# Reiniciar Servidor às 03:00 (n8n)

Workflow n8n que às 03:00 reinicia servidores Hetzner para clientes do PostgreSQL: obtém o servidor, executa poweroff, espera 2 minutos, aciona poweron e notifica no Chatwoot. 

## Visão Geral
Automatiza a reinicialização diário do servidores Hetzner para clientes listados no PostgreSQL, com notificação no Chatwoot. O workflow está em `Reiniciar Servidor as 03_00.json` e pode ser importado diretamente no n8n.

## Principais Funcionalidades
- Agendamento diário às 03:00 (`cron`: `0 3 * * *`).
- Seleção de clientes via Postgres com exclusões por `id`.
- Sequência de reboot: poweroff → espera 2 min → poweron.
- Notificação automática no Chatwoot por cliente processado.

## Fluxo do Workflow
- Schedule Trigger: agenda execução diária.
- Postgres (select): lê `public.cliente`, retornando todos os clientes exceto dois `id` específicos.
- Split In Batches: itera cliente a cliente.
- GetServer (Code): `GET https://api.hetzner.cloud/v1/servers` com `token_hetzner` do cliente; utiliza `servers[0]`.
- PowerOff (Code): `POST /servers/{id}/actions/poweroff`.
- Wait: aguarda 2 minutos.
- PowerOn (Code): `POST /servers/{id}/actions/poweron`.
- Chatwoot: envia mensagem na conversa `conversation_id` usando `{{ $json.nome }}`.

## Pré-requisitos
- n8n com permissão para importar workflows e configurar credenciais.
- PostgreSQL com tabela `public.cliente` contendo ao menos `id`, `nome`, `token_hetzner`.
- Tokens Hetzner Cloud válidos por cliente.
- Chatwoot com `account_id` e `conversation_id` válidos/acessíveis.
- Importação do **Community node** do chatwoot dentro do n8n `@devlikeapro/n8n-nodes-chatwoot.chatWoot`.

## Credenciais no n8n
- Postgres: credencial nomeada `Clientes` (usada no nó Postgres).
- Chatwoot: credencial nomeada `Chatwoot` (usada no nó Chatwoot).

## Como Importar e Configurar
1. n8n → Workflows → Import from File → selecione `Reiniciar Servidor as 03_00.json`.
2. Vincule credenciais:
   - Nó Postgres → `Clientes`.
   - Nó Chatwoot → `Chatwoot`.
3. Ative o workflow para iniciar a execução diária.

## Personalização
- Armarzenar dados sensíveis de forma segura (credenciais do servidor) no banco e importando quando necessário.
- Mensagem: personalize o template (ex.: incluir timestamp, `server_id`, status final).
---
Sinta-se à vontade para abrir issues ou PRs com melhorias e adaptações ao seu ambiente.
