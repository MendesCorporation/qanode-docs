# Administração

A seção de administração permite gerenciar usuários, papéis, permissões, configurações gerais, SMTP, alarmes, webhooks, auditoria e licenciamento.

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

[Convidando Usuários](../../assets/images/administracao-convidar-usuarios.mp4)

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

[Importação em lote](../../assets/images/administracao-importar-usuarios.mp4)

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
| **admin** | Todas as tabelas: Projects, Flows, Suites, Runs, Run Steps, Variables, Credentials, Users, Roles, Bugs, Audit Log |
| **architect** | Projects, Flows, Suites, Runs, Run Steps, Variables |
| **tester** | Projects, Flows, Suites, Runs, Run Steps |
| **none** | Nenhuma tabela — query builder desabilitado |

[Papéis Customizados](../../assets/images/administracao-papeis-customizados.mp4)

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

#### Componentes
| Permissão | Descrição |
|-----------|-----------|
| `component.view` | Visualizar componentes reutilizáveis |
| `component.create` | Criar e editar componentes próprios |
| `component.edit` | Editar componentes criados por outros usuários |
| `component.delete.own` | Excluir componentes próprios |
| `component.delete` | Excluir qualquer componente |
| `component.publish` | Publicar componentes para uso em cenários |

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

#### Defeitos — Enterprise
| Permissão | Descrição |
|-----------|-----------|
| `bug.view` | Visualizar defeitos — permite ver a lista, comentar e gerenciar anexos |
| `bug.create` | Abrir novos defeitos |
| `bug.edit` | Editar campos customizados do defeito |
| `bug.assign` | Atribuir defeitos a pessoas ou filas e fazer claim |
| `bug.transition` | Tramitar defeitos nas transições normais do workflow |
| `bug.transition_any` | Bypass administrativo — tramitar para qualquer status sem restrição de posse |
| `bug.run_sandbox` | Criar e descartar sandbox de investigação |
| `bug.delete_attachment_any` | Excluir anexos enviados por qualquer pessoa |
| `bug.configure_workflow` | Configurar o workflow, status, transições e campos customizados |

#### Configurações
| Permissão | Descrição |
|-----------|-----------|
| `settings.view` | Visualizar configurações — acesso básico à página de configurações |
| `settings.smtp` | Configurar SMTP — permite alterar as configurações de e-mail |
| `settings.integration_token` | Gerenciar os próprios tokens de integração |
| `settings.integration_token_all` | Ver todos os tokens, revogar tokens de outras pessoas e definir a política global de expiração |
| `settings.mfa` | Configurar MFA — permite gerenciar autenticação em dois fatores |
| `settings.audit` | Visualizar auditoria — acesso ao log de auditoria |
| `settings.report_template` | Gerenciar templates de relatório — permite criar e editar templates de PDF |

---

## Configurações Gerais — Super Admin

A aba **Geral** reúne configurações que afetam a instância como um todo. Ela fica disponível para Super Admins em **Configurações → Geral**.

As seções são organizadas em accordions para facilitar a navegação.

### Segurança e MFA

A configuração global de MFA fica em **Configurações → Geral → Segurança e MFA**.

Use essa seção para exigir autenticação em dois fatores e reforçar o acesso de usuários administrativos.

### Histórico de Versões — Enterprise

O histórico de versões permite manter snapshots de cenários e componentes.

| Configuração | Descrição |
|--------------|-----------|
| **Histórico de cenários** | Quantidade máxima de versões mantidas por cenário |
| **Histórico de componentes** | Quantidade máxima de versões mantidas por componente |

Como funciona:

- cenários criam snapshot ao salvar uma alteração relevante no fluxo;
- componentes criam snapshot ao publicar uma nova versão;
- versões antigas podem ser abertas em modo de consulta;
- a restauração cria uma nova versão atual a partir do snapshot escolhido;
- ao reduzir o limite, versões excedentes são removidas automaticamente.

[Histórico de Versões](../../assets/images/administracao-historico-versoes.mp4)

O padrão recomendado é manter versões suficientes para auditoria e reversão sem acumular histórico desnecessário.

### Retenção de Dados de Execução

A retenção de dados remove artefatos antigos de execução, como evidências e anexos técnicos de execuções antigas. Ela não remove o registro da execução em si.

Campos principais:

| Campo | Descrição |
|-------|-----------|
| **Ativar limpeza agendada** | Executa a limpeza automaticamente uma vez por dia |
| **Execuções com sucesso** | Dias para manter artefatos de execuções bem-sucedidas |
| **Execuções com falha** | Dias para manter artefatos de execuções com falha |
| **Execuções canceladas** | Dias para manter artefatos de execuções canceladas |
| **Execuções sandbox** | Dias para manter artefatos de execuções temporárias |
| **Manter últimas por fluxo** | Preserva as evidências mais recentes por cenário |

A limpeza agendada roda uma vez por dia, após o horário fixo definido pela instância. Também é possível usar **Pré-visualizar limpeza** para estimar o impacto antes de executar.

Por segurança, a limpeza preserva execuções vinculadas a defeitos ou tickets e não remove anexos de defeitos, evidências de tickets, sessões, templates de relatório ou sessões de navegador salvas.

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

## Tokens de Integração

Os **Tokens de Integração** permitem autenticar pipelines e automações sem depender de sessão de navegador.

Para acessar:

1. Vá em **Configurações** → **Tokens de Acesso**
2. Gere um token
3. Salve o valor como secret do pipeline
4. Use com `QANODE_URL` e `QANODE_TOKEN`

Usuários com `settings.integration_token_all` também podem:

- ver tokens de toda a instância
- revogar tokens de outras pessoas
- definir a política global de expiração

> Para o guia completo, veja [Integração CI/CD — Visão Geral](../ci-cd/visao-geral.md).

---

## Alarmes

Configure notificações automáticas para falhas em suítes:

1. Vá em **Configurações** → **Alarmes**
2. Configure:
   - **Suítes monitoradas** — Quais suítes disparam alarme
   - **Destinatários** — Quem recebe a notificação
   - **Condição** — Quando notificar (falha, sempre, etc.)

> **Requisito:** SMTP deve estar configurado para envio de alarmes por e-mail.

[Definir Alarmes](../../assets/images/administracao-definir-alarmes.mp4)

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

[Configurar Webhooks](../../assets/images/administracao-webhooks.mp4)

### Eventos Disponíveis

#### Execução de testes

| Evento | Descrição |
|--------|-----------|
| `run.completed` | Execução de cenário finalizada (sucesso ou falha) |
| `run.success` | Execução de cenário finalizada com sucesso |
| `run.failed` | Execução de cenário finalizada com falha |
| `suite.completed` | Execução de suíte finalizada (sucesso ou falha) |
| `suite.success` | Execução de suíte finalizada com sucesso |
| `suite.failed` | Execução de suíte finalizada com falha |

#### Defeitos

| Evento | Descrição |
|--------|-----------|
| `defect.opened` | Um novo defeito foi criado |
| `defect.status_changed` | Um defeito transitou para um status não-final do workflow |
| `defect.closed` | Um defeito transitou para um status final do workflow (corrigido, rejeitado, encerrado, etc.) |

### Escopo (Filtros)

Você pode filtrar os webhooks para disparar apenas em contextos específicos:

- **Projeto** — Apenas eventos de um projeto específico
- **Suíte** — Apenas execuções de uma suíte específica (somente eventos de execução de testes)
- **Cenário** — Apenas eventos de um cenário/fluxo específico

> Ao selecionar um projeto, as suítes e cenários são filtrados para mostrar apenas os daquele projeto. Ao selecionar uma suíte, o filtro de cenário é ocultado.

> **Observação:** O escopo de suíte não se aplica a eventos de defeito. Um webhook com escopo em uma suíte específica só dispara para eventos de execução de testes (`run.*`, `suite.*`), nunca para eventos de defeito.

### Payload do Webhook

Cada entrega é um `POST` HTTP com o seguinte corpo JSON:

```json
{
  "event": "<nome-do-evento>",
  "timestamp": "2026-04-15T10:30:00.000Z",
  "data": { ... }
}
```

#### Payload de execução de testes

```json
{
  "event": "run.completed",
  "timestamp": "2026-04-15T10:30:00.000Z",
  "data": {
    "runId": "uuid",
    "flowId": "uuid",
    "flowName": "Login Flow",
    "suiteId": "uuid",
    "suiteName": "Smoke Suite",
    "projectId": "uuid",
    "projectName": "App Mobile",
    "status": "success",
    "durationMs": 1234,
    "executor": "Nome do Usuário",
    "errorSummary": null
  }
}
```

#### Payload de defeitos (`defect.opened`, `defect.status_changed`, `defect.closed`)

```json
{
  "event": "defect.status_changed",
  "timestamp": "2026-04-15T15:10:00.000Z",
  "data": {
    "defectId": "uuid",
    "defectNumber": 42,
    "title": "Botão de login não responde no Safari",
    "description": "Ao clicar em Login no Safari 17...",
    "status": "in_progress",
    "previousStatus": "open",
    "severity": "high",
    "priority": "urgent",
    "resolution": null,
    "isFinal": false,
    "closedAt": null,
    "createdAt": "2026-04-15T14:32:00.000Z",
    "updatedAt": "2026-04-15T15:10:00.000Z",
    "project": { "id": "uuid", "name": "App Mobile" },
    "flow": { "id": "uuid", "name": "Login Flow" },
    "run": { "id": "uuid", "status": "failed" },
    "reporterUser": { "id": "uuid", "name": "Maria", "email": "maria@empresa.com" },
    "assigneeUser": { "id": "uuid", "name": "João Silva", "email": "joao@empresa.com" },
    "assigneeRole": null,
    "customFields": { "ambiente": "producao", "navegador": "Safari 17" },
    "actionComment": {
      "content": "Iniciando investigação no ambiente de staging",
      "user": { "id": "uuid", "name": "João Silva", "email": "joao@empresa.com" }
    }
  }
}
```

| Campo | Descrição |
|-------|-----------|
| `previousStatus` | Status antes desta transição; `null` em `defect.opened` |
| `isFinal` | `true` quando o status atual é final/encerrado |
| `closedAt` | Timestamp ISO de quando o defeito foi encerrado; `null` se ainda aberto |
| `customFields` | Mapa chave-valor dos campos personalizados definidos no workflow de defeitos |
| `actionComment` | Nota inserida durante esta ação, com o usuário que a realizou; `null` se nenhuma nota foi informada |

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

[Visualizar logs de auditoria](../../assets/images/administracao-visualizar-log-auditoria.mp4)


---

## Alteração de Senha

Qualquer usuário pode alterar sua própria senha:

1. Vá em **Configurações** → **Alterar Senha**
2. Informe a senha atual
3. Informe e confirme a nova senha
4. Clique em **Salvar**

[Alterar Senha](../../assets/images/administracao-alterar-senha.mp4)

---

## MFA (Autenticação em Dois Fatores)

Para maior segurança, ative a autenticação em dois fatores:

1. Vá em **Configurações** → **Geral** → **Segurança e MFA**
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

[Verificar Licença](../../assets/images/administracao-verificar-licenca.mp4)

---

## Dicas

- **Configure o SMTP primeiro** — muitas funcionalidades dependem de e-mail
- Use **papéis customizados** para controle fino de acesso
- Revise **Configurações → Geral** para MFA, histórico de versões e retenção de dados
- **Monitore a auditoria** regularmente para detectar ações inesperadas
- **Ative MFA** para contas de administrador
- **Desative usuários** em vez de excluir — preserva o histórico de auditoria
- **Use webhooks** para integrar o QANode com Slack, Teams, Discord ou qualquer serviço que aceite HTTP POST
- **Defina o secret** nos webhooks para garantir autenticidade das requisições
