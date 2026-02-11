# Variáveis

**Variáveis** são valores globais reutilizáveis em qualquer cenário de teste. Elas permitem centralizar configurações como URLs, credenciais de teste, flags e dados compartilhados.

---

## Gerenciando Variáveis

1. No menu lateral, clique em **Variáveis**
2. A lista exibe todas as variáveis cadastradas

[Lista de variáveis](../../assets/images/variavel.mp4) 
*Imagem: Tela de variáveis mostrando lista com nome, tipo, valor e última atualização*

---

## Criando uma Variável

1. Clique em **+ Nova Variável**
2. Preencha:

| Campo | Obrigatório | Descrição |
|-------|-------------|-----------|
| **Nome** | Sim | Identificador da variável (sem espaços) |
| **Tipo** | Sim | Tipo do valor |
| **Valor** | Sim | Valor da variável |
| **Secreta** | Não | Se ativado, o valor será criptografado |

3. Clique em **Salvar**

---

## Tipos de Variáveis

| Tipo | Descrição | Exemplo |
|------|-----------|---------|
| **String** | Texto | `https://api.exemplo.com` |
| **Number** | Número | `30000` |
| **Boolean** | Verdadeiro/Falso | `true` |
| **JSON** | Objeto ou array | `{ "timeout": 5000, "retries": 3 }` |

---

## Variáveis Secretas

Variáveis marcadas como **secretas** são criptografadas no banco de dados. São ideais para:

- Senhas de teste
- Tokens de API
- Chaves de acesso

Características:
- O valor é **mascarado** na interface (exibido como `••••••`)
- O valor é **criptografado** no banco de dados
- O valor real é **acessível** durante a execução dos fluxos

> **Nota:** Para credenciais de conexão (banco de dados, SSH, API), prefira usar [Credenciais](../credenciais/credenciais.md) em vez de variáveis secretas.

---

## Usando Variáveis nos Fluxos

Acesse variáveis usando a sintaxe de expressões:

```
{{ variables.NOME_DA_VARIAVEL }}
```

### Exemplos de Uso

**URL de API:**
```
URL: {{ variables.API_URL }}/api/users
```

**Timeout:**
```
Timeout: {{ variables.DEFAULT_TIMEOUT }}
```

**Credenciais de teste:**
```
Email: {{ variables.TEST_EMAIL }}
Senha: {{ variables.TEST_PASSWORD }}
```

**Configuração JSON:**
```
Retries: {{ variables.CONFIG.retries }}
Timeout: {{ variables.CONFIG.timeout }}
```

---

## Sobrescrita em Tempo de Execução

O nó **Set Variable** pode sobrescrever variáveis durante a execução:

```
[Set Variable: API_URL = https://staging.exemplo.com]
    │
    ▼
[HTTP Request: GET {{ variables.API_URL }}/api/health]
```

A sobrescrita é temporária — afeta apenas a execução atual. O valor original no banco de dados não é alterado.

---

## Boas Práticas

### Nomenclatura

Use **UPPER_SNAKE_CASE** para variáveis globais:

| Bom | Ruim |
|-----|------|
| `API_URL` | `apiUrl` |
| `TEST_EMAIL` | `email` |
| `DEFAULT_TIMEOUT` | `timeout` |
| `MAX_RETRIES` | `retries` |

### Organização por Ambiente

Crie variáveis com prefixo de ambiente para facilitar a troca:

```
PROD_API_URL = https://api.exemplo.com
STAGING_API_URL = https://staging.exemplo.com
DEV_API_URL = http://localhost:3000
```

### Quando Usar Variáveis vs Credenciais

| Cenário | Use |
|---------|-----|
| URL base de API | Variável |
| Token de autenticação | Credencial HTTP |
| Dados de conexão ao banco | Credencial de banco |
| Senha de teste genérica | Variável secreta |
| Flag de configuração | Variável |
| Dados de conexão SSH | Credencial SSH |

---

## Dicas

- **Centralize URLs** em variáveis para facilitar troca de ambiente
- **Use variáveis secretas** para dados sensíveis
- **Prefira credenciais** para dados de conexão estruturados
- Variáveis são **globais** — acessíveis em qualquer fluxo
- O tipo **JSON** é útil para agrupar configurações relacionadas
