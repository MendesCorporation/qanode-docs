# Administração

A seção de administração permite gerenciar usuários, papéis, permissões, configurações de SMTP, alarmes, auditoria e licenciamento.

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
3. Defina o nome e selecione as permissões

### Lista Completa de Permissões

#### Projetos
| Permissão | Descrição |
|-----------|-----------|
| `project.create` | Criar projetos |
| `project.edit.own` | Editar projetos próprios |
| `project.edit.all` | Editar qualquer projeto |
| `project.archive` | Arquivar projetos |

#### Fluxos (Cenários)
| Permissão | Descrição |
|-----------|-----------|
| `flow.create` | Criar cenários |
| `flow.edit.own` | Editar cenários próprios |
| `flow.edit.all` | Editar qualquer cenário |
| `flow.delete.own` | Excluir cenários próprios |
| `flow.delete.all` | Excluir qualquer cenário |
| `flow.run` | Executar cenários |

#### Suítes
| Permissão | Descrição |
|-----------|-----------|
| `suite.create` | Criar suítes |
| `suite.edit` | Editar suítes |
| `suite.delete` | Excluir suítes |
| `suite.run` | Executar suítes |

#### Variáveis
| Permissão | Descrição |
|-----------|-----------|
| `variable.create` | Criar variáveis |
| `variable.edit` | Editar variáveis |
| `variable.delete` | Excluir variáveis |

#### Credenciais
| Permissão | Descrição |
|-----------|-----------|
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

#### Dashboard
| Permissão | Descrição |
|-----------|-----------|
| `dashboard.create` | Criar dashboards |
| `dashboard.edit` | Editar dashboards |
| `dashboard.delete` | Excluir dashboards |
| `dashboard.share` | Compartilhar dashboards |
| `dashboard.sql` | Usar SQL no query builder |

#### Usuários
| Permissão | Descrição |
|-----------|-----------|
| `user.view` | Visualizar lista de usuários |
| `user.create` | Convidar usuários |
| `user.edit` | Editar usuários |
| `user.delete` | Excluir usuários |

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
| `settings.smtp` | Configurar SMTP |
| `settings.mfa` | Configurar MFA |
| `settings.audit` | Visualizar auditoria |
| `settings.report_template` | Configurar template de relatório |
| `settings.license` | Gerenciar licença |

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

<!-- ![Configuração SMTP](../../assets/images/smtp-config.png) -->
*Imagem: Formulário de SMTP com campos preenchidos e indicador de status*

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
