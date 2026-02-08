# Credenciais

**Credenciais** armazenam informações de conexão de forma segura e centralizada. Em vez de colocar senhas e tokens diretamente nos nós, você cria uma credencial e a referencia nos fluxos.

---

## Gerenciando Credenciais

1. No menu lateral, clique em **Credenciais**
2. A lista exibe todas as credenciais cadastradas

<!-- ![Lista de credenciais](../../assets/images/lista-credenciais.png) -->
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

## Testando Conexão

Antes de usar uma credencial, teste a conexão:

1. Na tela de edição da credencial, clique em **Testar Conexão**
2. O QANode tentará se conectar usando os dados fornecidos
3. O resultado será exibido: sucesso ✅ ou erro ❌ com mensagem detalhada

<!-- ![Teste de conexão](../../assets/images/testar-conexao.png) -->
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
