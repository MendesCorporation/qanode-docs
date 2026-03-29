# Credenciais

**Credenciais** armazenam informações de conexão de forma segura e centralizada. Em vez de colocar senhas e tokens diretamente nos nós, você cria uma credencial e a referencia nos fluxos.

---

## Gerenciando Credenciais

1. No menu lateral, clique em **Credenciais**
2. A lista exibe todas as credenciais cadastradas

[Lista de credenciais](../../assets/images/lista-credencial.mp4) 
*Imagem: Tela de credenciais com lista mostrando nome, tipo e data de criação*

---

## Criando uma Credencial

1. Clique em **+ Nova Credencial**
2. Selecione o **tipo** de credencial
3. Preencha os campos específicos do tipo
4. (Opcional) Clique em **Testar Conexão** para verificar
5. Clique em **Salvar**

---

## Tipos de Credenciais

### HTTP / API

Para conexões com APIs REST.

| Campo | Descrição |
|-------|-----------|
| **Nome** | Nome da credencial |
| **URL Base** | URL base da API (ex: `https://api.exemplo.com`) |
| **Tipo de Auth** | `None`, `Bearer Token`, `Basic Auth` |
| **Token** | Token Bearer (quando Bearer) |
| **Usuário** | Nome de usuário (quando Basic) |
| **Senha** | Senha (quando Basic) |
| **Headers** | Headers customizados (chave-valor) |

**Uso no nó HTTP Request:** Selecione a credencial e informe apenas o path:
```
Credencial: "API Exemplo"
URL: /api/users  (o URL base é adicionado automaticamente)
```

### Override por Pipeline — Enterprise

Na integração de CI/CD do QANode Enterprise, campos específicos de uma credencial podem ser sobrescritos apenas para a execução atual.

Exemplo:

```bash
npx @qanode/cli run scenario \
  --scenario-id SCENARIO_ID \
  --credential api-main.token=$API_TOKEN \
  --wait
```

Esse padrão é útil para:

- tokens efêmeros de pipeline
- segredos temporários de homologação
- endpoints de ambiente transitório

O valor salvo na credencial continua intacto após o término da execução.

> Para detalhes, veja [Overrides por Execução](../ci-cd/overrides.md).

---

### PostgreSQL

| Campo | Descrição |
|-------|-----------|
| **Nome** | Nome da credencial |
| **Host** | Endereço do servidor |
| **Porta** | Porta (padrão: 5432) |
| **Banco de Dados** | Nome do banco |
| **Usuário** | Usuário de conexão |
| **Senha** | Senha de conexão |
| **SSL** | Usar conexão SSL |

---

### MySQL

| Campo | Descrição |
|-------|-----------|
| **Nome** | Nome da credencial |
| **Host** | Endereço do servidor |
| **Porta** | Porta (padrão: 3306) |
| **Banco de Dados** | Nome do banco |
| **Usuário** | Usuário de conexão |
| **Senha** | Senha de conexão |
| **SSL** | Usar conexão SSL |

---

### MariaDB

Mesmos campos do MySQL.

---

### Oracle

| Campo | Descrição |
|-------|-----------|
| **Nome** | Nome da credencial |
| **Host** | Endereço do servidor |
| **Porta** | Porta (padrão: 1521) |
| **Service Name** | Nome do serviço Oracle |
| **Usuário** | Usuário de conexão |
| **Senha** | Senha de conexão |

---

### MongoDB

| Campo | Descrição |
|-------|-----------|
| **Nome** | Nome da credencial |
| **URI** | Connection string MongoDB |
| **Banco de Dados** | Nome do banco (quando não incluído na URI) |

**URI de exemplo:**
```
mongodb://usuario:senha@host:27017/banco?authSource=admin
mongodb+srv://usuario:senha@cluster.mongodb.net/banco
```

---

### SSH

| Campo | Descrição |
|-------|-----------|
| **Nome** | Nome da credencial |
| **Host** | Endereço do servidor |
| **Porta** | Porta SSH (padrão: 22) |
| **Usuário** | Nome de usuário |
| **Senha** | Senha (autenticação por senha) |
| **Chave Privada** | Conteúdo da chave privada (autenticação por chave) |
| **Passphrase** | Frase-senha da chave (se protegida) |

---

### Email

Credencial para conexão com caixas de e-mail via IMAP. Usada pelo nó [Email Inbox](../nos/utilitarios/email-inbox.md).

| Campo | Descrição |
|-------|-----------|
| **Nome** | Nome da credencial |
| **Provedor** | `Gmail`, `Outlook` ou `Custom IMAP` |
| **Auth Mode** | `Password / App Password` ou `OAuth2` |
| **Email** | Endereço de e-mail da conta |
| **IMAP Host** | Servidor IMAP (preenchido automaticamente para Gmail e Outlook) |
| **IMAP Port** | Porta IMAP (padrão: 993) |
| **Secure** | Usar TLS/SSL (padrão: true) |

**Modo Password:**

| Campo | Descrição |
|-------|-----------|
| **Password / App Password** | Senha da conta ou App Password gerada pelo provedor |

> Para Gmail com senha, é obrigatório usar uma **App Password** — não a senha normal da conta. [Gerar App Password](https://myaccount.google.com/apppasswords)

**Modo OAuth2:**

| Campo | Descrição |
|-------|-----------|
| **Client ID** | ID do cliente OAuth registrado no provedor |
| **Client Secret** | Segredo do cliente OAuth |
| **Tenant ID** | Apenas para Outlook — ID do tenant Azure (padrão: `common`) |
| **Scopes** | Escopos OAuth (opcional; padrão preenchido automaticamente) |
| **OAuth Callback URI** | URI que deve ser cadastrada no console do provedor (somente leitura) |

Após preencher Client ID e Secret, clique em **Connect OAuth (Browser)** para autorizar. Uma janela abrirá para login no provedor e, ao concluir, o Access Token e Refresh Token serão preenchidos automaticamente.

> **Gmail:** cadastre o **OAuth Callback URI** exibido no campo como "URI de redirecionamento autorizado" no [Google Cloud Console](https://console.cloud.google.com/) → Credenciais → OAuth 2.0.

> **Outlook:** cadastre o URI como "Redirect URI" no [Azure Portal](https://portal.azure.com/) → Registros de aplicativo → Autenticação.

---

## Testando Conexão

Antes de usar uma credencial, teste a conexão:

1. Na tela de edição da credencial, clique em **Testar Conexão**
2. O QANode tentará se conectar usando os dados fornecidos
3. O resultado será exibido: sucesso ✅ ou erro ❌ com mensagem detalhada

[Teste de conexão](../../assets/images/teste-credencial.mp4)
*Imagem: Credencial com botão de teste de conexão e mensagem de sucesso*

---

## Usando Credenciais nos Fluxos

### Em Nós de Banco de Dados

No nó PostgreSQL, MySQL, MariaDB, Oracle ou MongoDB:

1. No campo **Credencial**, selecione a credencial
2. Os campos de conexão são preenchidos automaticamente
3. Configure apenas a query/operação

### Em Nós HTTP Request

1. No campo **Credencial**, selecione a credencial HTTP
2. A URL base e a autenticação são aplicadas automaticamente
3. No campo **URL**, informe apenas o path

### Em Nós SSH

1. No campo **Credencial**, selecione a credencial SSH
2. Os dados de conexão são aplicados automaticamente
3. Configure apenas os comandos

---

## Segurança

- Senhas e tokens são **criptografados** no banco de dados
- Valores sensíveis são **mascarados** na interface
- Credenciais são acessíveis somente por usuários com permissão
- O proprietário da credencial pode editá-la; outros precisam de permissão de admin

---

## Dicas

- **Crie credenciais por ambiente** — Produção, Staging, Dev
- **Sempre teste** a conexão antes de usar no fluxo
- **Use nomes descritivos**: "PostgreSQL Produção" é melhor que "pg1"
- **Prefira credenciais** em vez de colocar senhas diretamente nos nós
- Ao **duplicar ambientes**, crie novas credenciais para cada um
