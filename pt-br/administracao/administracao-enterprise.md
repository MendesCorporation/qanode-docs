# Administração

A seção de administração permite gerenciar usuários, papéis, permissões, configurações de SMTP, alarmes, webhooks, auditoria e licenciamento.

Acesse através do ícone de **engrenagem** (⚙️) no menu lateral → **Configurações**.

---

## Usuários

### Convidando Usuários

1. Vá em **Configurações** → **Usuários**
2. Clique em **Convidar Usuário**
3. Preencha:
   - **E-mail**: E-mail do novo usuário
   - **Papel**: Papel atribuído (define permissões)
4. Um e-mail de convite será enviado com link de ativação

> **Requisito:** SMTP deve estar configurado para envio de convites.

### Importação em Lote

Para cadastrar vários usuários de uma vez:

1. Vá em **Configurações** → **Usuários**
2. Clique no ícone de **importação** (ao lado do botão de convidar)
3. Baixe o **template Excel** clicando em **Download Template**
4. Preencha a planilha com os dados dos usuários:

| Coluna | Descrição |
|--------|-----------|
| **name** | Nome completo do usuário |
| **email** | E-mail do usuário |
| **role** | Nome do papel (ex: Admin, Architect, Tester) |

5. Faça upload do arquivo Excel (.xlsx ou .xls) arrastando para a área indicada ou clicando para selecionar
6. O sistema processará cada linha e enviará convites por e-mail

Após o processamento, um resumo é exibido com:
- Quantidade de usuários importados com sucesso
- Lista de falhas com o número da linha, e-mail e motivo do erro (ex: e-mail já existe, papel não encontrado, campos obrigatórios faltando)

> **Nota:** A importação respeita o limite de usuários da licença. Se o arquivo contiver mais usuários do que o plano permite, o sistema informará e oferecerá a opção de importar apenas até o limite disponível.

### Gerenciando Usuários

Na lista de usuários você pode:

| Ação | Descrição |
|------|-----------|
| **Ativar/Desativar** | Ativa ou desativa o acesso do usuário |
| **Alterar Papel** | Muda o papel (e permissões) do usuário |
| **Excluir** | Remove o usuário permanentemente |

### Status de Usuários

| Status | Descrição |
|--------|-----------|
| **Ativo** | Usuário com acesso normal |
| **Inativo** | Acesso bloqueado |
| **Pendente** | Convite enviado, aguardando ativação |

---

## Papéis e Permissões

### Papéis do Sistema

O QANode inclui papéis pré-definidos que não podem ser removidos:

| Papel | Descrição |
|-------|-----------|
| **Super Admin** | Acesso total a todas as funcionalidades |
| **Admin** | Acesso administrativo |
| **Architect** | Acesso de adminsitração de projetos |
| **Tester** | Acesso de cenários e execuções |

### Papéis Customizados

Crie papéis com permissões específicas:

1. Vá em **Configurações** → **Papéis e Permissões**
2. Clique em **+ Novo Papel**
3. Defina o nome, a visão de tabelas e selecione as permissões

### Visão de Tabelas (Table View)

Ao criar ou editar um papel, o campo **Visão de Tabelas** define quais tabelas do banco de dados ficam acessíveis no **query builder** dos dashboards personalizados. Isso controla o que o usuário pode consultar ao criar widgets com queries SQL ou ao explorar dados nos dashboards.

| Perfil | Tabelas acessíveis |
|--------|-------------------|
| **admin** | Todas as tabelas: Projects, Flows, Suites, Runs, Run Steps, Variables, Credentials, Users, Roles, Audit Log |
| **architect** | Projects, Flows, Suites, Runs, Run Steps, Variables |
| **tester** | Projects, Flows, Suites, Runs, Run Steps |
| **none** | Nenhuma tabela — query builder desabilitado |

> **Nota:** Colunas sensíveis (senhas, tokens, dados criptografados) são automaticamente ocultadas, independentemente do perfil de visão.

#### Perfis padrão dos papéis do sistema

| Papel do Sistema | Visão de Tabelas |
|------------------|-----------------|
| **Super Admin** | admin |
| **Admin** | admin |
| **Architect** | architect |
| **Tester** | tester |

### Lista Completa de Permissões

#### Cenários (Flows)
| Permissão | Descrição |
|-----------|-----------|
| `flow.view` | Visualizar cenários — permite ver a lista e detalhes dos cenários |
| `flow.create` | Criar cenários |
| `flow.edit.own` | Editar cenários próprios (criados pelo usuário) |
| `flow.edit.all` | Editar qualquer cenário (incluindo de outros usuários) |
| `flow.delete.own` | Excluir cenários próprios |
| `flow.delete.all` | Excluir qualquer cenário |
| `flow.run` | Executar cenários |

#### Projetos
| Permissão | Descrição |
|-----------|-----------|
| `project.view` | Visualizar projetos — permite ver a lista e detalhes dos projetos |
| `project.create` | Criar projetos |
| `project.edit.own` | Editar projetos próprios (criados pelo usuário) |
| `project.edit.all` | Editar qualquer projeto |
| `project.archive` | Arquivar projetos |
| `project.delete` | Excluir projetos permanentemente |

#### Suítes
| Permissão | Descrição |
|-----------|-----------|
| `suite.view` | Visualizar suítes — permite ver a lista e detalhes das suítes |
| `suite.create` | Criar suítes |
| `suite.edit` | Editar suítes |
| `suite.delete` | Excluir suítes |
| `suite.run` | Executar suítes |

#### Execuções (Runs)
| Permissão | Descrição |
|-----------|-----------|
| `run.view` | Visualizar execuções próprias — permite ver o histórico das execuções realizadas pelo usuário |
| `run.view.all` | Visualizar todas as execuções — permite ver execuções de todos os usuários |
| `run.cancel` | Cancelar execuções em andamento |
| `run.delete` | Excluir registros de execução |

#### Variáveis
| Permissão | Descrição |
|-----------|-----------|
| `variable.view` | Visualizar variáveis — permite ver a lista de variáveis |
| `variable.create` | Criar variáveis |
| `variable.edit` | Editar variáveis |
| `variable.delete` | Excluir variáveis |
| `variable.view.secret` | Visualizar valores secretos — permite revelar o valor de variáveis marcadas como secretas |

#### Credenciais
| Permissão | Descrição |
|-----------|-----------|
| `credential.view` | Visualizar credenciais — permite ver a lista de credenciais cadastradas |
| `credential.create` | Criar credenciais |
| `credential.edit` | Editar credenciais |
| `credential.delete` | Excluir credenciais |

#### Provedores
| Permissão | Descrição |
|-----------|-----------|
| `provider.view` | Visualizar provedores |
| `provider.create` | Registrar provedores |
| `provider.edit` | Editar provedores |
| `provider.delete` | Remover provedores |

#### Relatórios (Reports)
| Permissão | Descrição |
|-----------|-----------|
| `report.view` | Visualizar relatórios — permite acessar os relatórios PDF gerados nas execuções |
| `report.export` | Exportar relatórios — permite baixar relatórios em PDF |

#### Dashboards
| Permissão | Descrição |
|-----------|-----------|
| `dashboard.view` | Visualizar dashboards — permite acessar e ver dashboards existentes |
| `dashboard.create` | Criar dashboards personalizados |
| `dashboard.edit` | Editar dashboards |
| `dashboard.delete` | Excluir dashboards |
| `dashboard.share` | Compartilhar dashboards — permite tornar dashboards públicos ou compartilhar com papéis específicos |
| `dashboard.sql` | Executar SQL — permite usar consultas SQL diretas no query builder de widgets |

#### Webhooks
| Permissão | Descrição |
|-----------|-----------|
| `webhook.view` | Visualizar webhooks |
| `webhook.create` | Criar webhooks |
| `webhook.edit` | Editar webhooks |
| `webhook.delete` | Excluir webhooks |

#### Usuários
| Permissão | Descrição |
|-----------|-----------|
| `user.view` | Visualizar lista de usuários |
| `user.invite` | Convidar novos usuários — envia e-mail de convite com link de ativação |
| `user.edit` | Editar dados de usuários (alterar papel, informações) |
| `user.delete` | Excluir usuários permanentemente |
| `user.deactivate` | Desativar/ativar usuários — bloqueia ou libera o acesso sem excluir |

#### Papéis
| Permissão | Descrição |
|-----------|-----------|
| `role.view` | Visualizar papéis |
| `role.create` | Criar papéis |
| `role.edit` | Editar papéis |
| `role.delete` | Excluir papéis |

#### Configurações
| Permissão | Descrição |
|-----------|-----------|
| `settings.view` | Visualizar configurações — acesso básico à página de configurações |
| `settings.smtp` | Configurar SMTP — permite alterar as configurações de e-mail |
| `settings.mfa` | Configurar MFA — permite gerenciar autenticação em dois fatores |
| `settings.audit` | Visualizar auditoria — acesso ao log de auditoria |
| `settings.report_template` | Gerenciar templates de relatório — permite criar e editar templates de PDF |

---

## Configuração SMTP

Para envio de e-mails (convites, relatórios, alarmes):

1. Vá em **Configurações** → **SMTP**
2. Preencha:

| Campo | Descrição |
|-------|-----------|
| **Host** | Servidor SMTP (ex: `smtp.gmail.com`) |
| **Porta** | Porta do servidor |
| **Segurança** | STARTTLS (587), TLS/SSL (465) ou Nenhuma |
| **Usuário** | E-mail/usuário de autenticação |
| **Senha** | Senha do e-mail |
| **Remetente** | Endereço de envio (from) |

3. Clique em **Salvar** — o sistema testa a conexão automaticamente

---

## Alarmes

Configure notificações automáticas para falhas em suítes:

1. Vá em **Configurações** → **Alarmes**
2. Configure:
   - **Suítes monitoradas** — Quais suítes disparam alarme
   - **Destinatários** — Quem recebe a notificação
   - **Condição** — Quando notificar (falha, sempre, etc.)

> **Requisito:** SMTP deve estar configurado para envio de alarmes por e-mail.

---

## Webhooks

Webhooks permitem enviar notificações HTTP POST automaticamente para serviços externos quando eventos de execução ocorrem. Cada usuário gerencia seus próprios webhooks de forma independente.

### Configurando Webhooks

1. Vá em **Configurações** → **Webhooks**
2. Clique em **+ Novo Webhook**
3. Preencha:

| Campo | Descrição |
|-------|-----------|
| **Nome** | Nome identificador do webhook |
| **URL** | Endereço HTTP/HTTPS que receberá o POST |
| **Secret** | (Opcional) Chave HMAC para assinatura do payload |
| **Eventos** | Quais eventos disparam o webhook |
| **Escopo** | Filtro por projeto, suíte ou cenário específico |

### Eventos Disponíveis

| Evento | Descrição |
|--------|-----------|
| `run.completed` | Execução de cenário finalizada (sucesso ou falha) |
| `run.success` | Execução de cenário finalizada com sucesso |
| `run.failed` | Execução de cenário finalizada com falha |
| `suite.completed` | Execução de suíte finalizada (sucesso ou falha) |
| `suite.success` | Execução de suíte finalizada com sucesso |
| `suite.failed` | Execução de suíte finalizada com falha |

### Escopo (Filtros)

Você pode filtrar os webhooks para disparar apenas em contextos específicos:

- **Projeto** — Apenas execuções de um projeto específico
- **Suíte** — Apenas execuções de uma suíte específica
- **Cenário** — Apenas execuções de um cenário específico

> Ao selecionar um projeto, as suítes e cenários são filtrados para mostrar apenas os daquele projeto. Ao selecionar uma suíte, o filtro de cenário é ocultado.

### Payload do Webhook

O corpo do POST enviado contém:

```json
{
  "event": "run.completed",
  "timestamp": "2026-02-13T10:30:00.000Z",
  "data": {
    "runId": "uuid",
    "flowId": "uuid",
    "flowName": "Nome do Cenário",
    "suiteId": "uuid",
    "suiteName": "Nome da Suíte",
    "projectId": "uuid",
    "projectName": "Nome do Projeto",
    "status": "success",
    "durationMs": 1234,
    "executor": "Nome do Usuário",
    "errorSummary": null
  }
}
```

### Assinatura HMAC

Se o campo **Secret** for preenchido, o QANode adiciona o header `X-QANode-Signature` com a assinatura HMAC-SHA256 do payload. Isso permite que o serviço receptor valide a autenticidade da requisição.

Exemplo de header:
```
X-QANode-Signature: sha256=abc123...
```

### Retentativas

Em caso de falha na entrega (erro de rede ou código HTTP diferente de 2xx), o QANode tenta reenviar automaticamente até **3 vezes** com intervalos crescentes:

| Tentativa | Intervalo |
|-----------|-----------|
| 1ª | Imediata |
| 2ª | 1 segundo |
| 3ª | 5 segundos |

### Log de Entregas

Para cada webhook, é possível visualizar o histórico das últimas 50 entregas:

1. Clique no ícone de **lista** ao lado do webhook
2. Veja o status de cada entrega (OK/Falhou), código HTTP, tentativas e erros

### Testando Webhooks

Clique no ícone de **play** ao lado do webhook para enviar um payload de teste. Isso permite verificar se a URL está acessível e respondendo corretamente.

### Permissões

Os seguintes papéis possuem acesso a webhooks:

| Papel | Acesso |
|-------|--------|
| **Super Admin** | Total |
| **Admin** | Total |
| **Architect** | Total |
| **Tester** | Sem acesso |

> Cada usuário visualiza e gerencia apenas seus próprios webhooks.

---

## Auditoria

O log de auditoria registra todas as ações importantes no sistema:

### Ações Rastreadas

| Ação | Descrição |
|------|-----------|
| **create** | Criação de recurso |
| **update** | Atualização de recurso |
| **delete** | Exclusão de recurso |
| **execute** | Execução de cenário/suíte |
| **login** | Login de usuário |
| **logout** | Logout de usuário |
| **import** | Importação de dados |
| **export** | Exportação de dados |
| **test** | Teste de conexão |
| **invite** | Convite de usuário |
| **setup** | Configuração inicial |

### Entidades Rastreadas

Projetos, Fluxos, Suítes, Execuções, Variáveis, Credenciais, Provedores, Usuários, Papéis, Configurações, Sistema.

### Informações do Log

| Campo | Descrição |
|-------|-----------|
| **Ação** | Tipo de ação realizada |
| **Entidade** | Tipo e nome do recurso afetado |
| **Usuário** | Quem realizou a ação |
| **IP** | Endereço IP de origem |
| **Data/Hora** | Quando a ação ocorreu |

### Visualizando

1. Vá em **Configurações** → **Auditoria**
2. Navegue pelo histórico usando paginação
3. Use filtros para buscar ações específicas


---

## Alteração de Senha

Qualquer usuário pode alterar sua própria senha:

1. Vá em **Configurações** → **Alterar Senha**
2. Informe a senha atual
3. Informe e confirme a nova senha
4. Clique em **Salvar**

---

## MFA (Autenticação em Dois Fatores)

Para maior segurança, ative a autenticação em dois fatores:

1. Vá em **Configurações** → **MFA**
2. Escaneie o QR Code com um app autenticador (Google Authenticator, Authy, etc.)
3. Informe o código gerado para confirmar
4. O MFA estará ativo — códigos serão solicitados em cada login

---

## Licença

Gerencie a licença do QANode:

1. Vá em **Configurações** → **Licença**
2. Visualize:
   - **Status** da licença (válida/expirada)
   - **Data de expiração**
   - **Dias restantes**
   - **Limite de usuários**
   - **Usuários ativos** vs limite

---

## Dicas

- **Configure o SMTP primeiro** — muitas funcionalidades dependem de e-mail
- Use **papéis customizados** para controle fino de acesso
- **Monitore a auditoria** regularmente para detectar ações inesperadas
- **Ative MFA** para contas de administrador
- **Desative usuários** em vez de excluir — preserva o histórico de auditoria
- **Use webhooks** para integrar o QANode com Slack, Teams, Discord ou qualquer serviço que aceite HTTP POST
- **Defina o secret** nos webhooks para garantir autenticidade das requisições
