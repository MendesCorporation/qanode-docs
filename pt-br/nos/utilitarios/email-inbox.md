# Nó Email Inbox

O nó **Email Inbox** conecta a uma caixa de e-mail via IMAP e aguarda ou busca mensagens que correspondam a critérios definidos. É ideal para validar recebimento de e-mails, extrair OTPs e capturar links de confirmação em fluxos de teste.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `email-inbox` |
| **Categoria** | Utilitários |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Pré-requisito: Credencial de E-mail

Antes de usar o nó, crie uma **Credencial de E-mail** na seção [Credenciais](../../credenciais/credenciais.md#email). Ela armazena a conexão IMAP e a autenticação de forma segura.

---

## Campos de Configuração

### Geral

| Campo | Tipo | Padrão | Descrição |
|-------|------|--------|-----------|
| **Credencial** | seleção | — | Credencial de e-mail a usar |
| **Operação** | seleção | `Extract Email` | O que fazer ao encontrar o e-mail |
| **Pasta** | texto | `INBOX` | Pasta IMAP a monitorar |
| **Apenas Não Lidos** | boolean | `true` | Busca somente mensagens não lidas |
| **Marcar Como Lido** | boolean | `false` | Marca as mensagens encontradas como lidas |

### Operações disponíveis

| Valor | Descrição |
|-------|-----------|
| `Extract Email` | Aguarda e retorna o e-mail completo |
| `Extract OTP` | Extrai um código numérico do corpo do e-mail |
| `Extract Link` | Extrai uma URL do corpo do e-mail |

### Filtros de busca

Todos os filtros são opcionais. Quando preenchido, o campo deve estar **contido** no e-mail para que ele seja considerado.

| Campo | Descrição | Exemplo |
|-------|-----------|---------|
| **From Contains** | Filtra por remetente | `noreply@gmail.com` |
| **To Contains** | Filtra por destinatário | `meuemail@` |
| **Subject Contains** | Filtra por assunto | `Código de verificação` |
| **Body Contains** | Filtra por texto no corpo | `seu código é` |

### Filtro de data

| Campo | Opções | Descrição |
|-------|--------|-----------|
| **Received After Mode** | `Step Start` / `Absolute Date/Time` / `No Date Filter` | Modo do filtro de data |
| **Received After** | ISO 8601 | Usado apenas quando modo = `Absolute Date/Time` |

> **Step Start (recomendado):** o nó usa o UID do próximo e-mail no momento em que começa a execução, ignorando mensagens anteriores de forma confiável, independente de fuso horário.

### Polling e limites

| Campo | Tipo | Padrão | Descrição |
|-------|------|--------|-----------|
| **Timeout (ms)** | number | `120000` | Tempo máximo de espera (0 = sem espera, busca uma vez) |
| **Poll Interval (ms)** | number | `3000` | Intervalo entre verificações |
| **Max Scan** | number | `30` | Máximo de e-mails a analisar por ciclo |
| **Max Matches** | number | `1` | Máximo de e-mails a retornar |

### Extração de OTP (quando operação = `Extract OTP`)

| Campo | Tipo | Padrão | Descrição |
|-------|------|--------|-----------|
| **OTP Regex (opcional)** | regex | automático | Expressão personalizada para capturar o código |
| **OTP Min Length** | number | `6` | Comprimento mínimo do código |
| **OTP Max Length** | number | `6` | Comprimento máximo do código |

Quando nenhum regex é informado, o nó procura automaticamente por sequências numéricas com o comprimento configurado.

### Extração de Link (quando operação = `Extract Link`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Link Domain Contains** | texto | Filtra links pelo domínio (ex: `accounts.google.com`) |
| **Link Regex (opcional)** | regex | Expressão personalizada para capturar a URL |

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `found` | `boolean` | Se algum e-mail foi encontrado |
| `matchCount` | `number` | Quantidade de e-mails que corresponderam |
| `attempts` | `number` | Número de verificações realizadas |
| `email` | `object` | Primeiro e-mail encontrado (objeto completo) |
| `emails` | `array` | Todos os e-mails encontrados (até `maxMatches`) |
| `links` | `array` | URLs encontradas no primeiro e-mail |
| `otp` | `string` | Código extraído (apenas em `extractOtp`) |
| `link` | `string` | URL extraída (apenas em `extractLink`) |

### Estrutura do objeto `email`

```json
{
  "uid": 12345,
  "messageId": "<abc@gmail.com>",
  "subject": "Seu código de verificação",
  "from": "Empresa <noreply@empresa.com>",
  "to": "usuario@gmail.com",
  "date": "2026-03-08T12:00:00.000Z",
  "text": "Seu código é 482910. Válido por 10 minutos.",
  "html": "<p>Seu código é <strong>482910</strong>...</p>",
  "links": ["https://empresa.com/confirmar?token=xyz"]
}
```

---

## Exemplos Práticos

### Aguardar e-mail de confirmação de cadastro

```
[Clique em "Criar conta" no navegador]
    │
    ▼
[Email Inbox]
  Operação: Extract Email
  Subject Contains: "Confirme seu e-mail"
  From Contains: noreply@servico.com
  Timeout: 60000ms
    │
    ▼
[HTTP Request: GET {{ steps.emailInbox.outputs.links[0] }}]
```

### Extrair OTP de 6 dígitos

```
[Clique em "Enviar código" no app]
    │
    ▼
[Email Inbox]
  Operação: Extract OTP
  Subject Contains: "código de verificação"
  OTP Min Length: 6
  OTP Max Length: 6
  Timeout: 30000ms
    │
    ▼
[Type no campo OTP: {{ steps.emailInbox.outputs.otp }}]
```

### Extrair link de redefinição de senha

```
[Clique em "Esqueci minha senha"]
    │
    ▼
[Email Inbox]
  Operação: Extract Link
  Subject Contains: "redefinir senha"
  Link Domain Contains: accounts.empresa.com
  Timeout: 30000ms
    │
    ▼
[Navigate: {{ steps.emailInbox.outputs.link }}]
```

---

## Configuração por Provedor

### Gmail

| Modo | Configuração necessária |
|------|------------------------|
| **Senha / App Password** | Ative "Acesso a app menos seguro" ou gere uma [App Password](https://myaccount.google.com/apppasswords) |
| **OAuth2** | Crie um app no Google Cloud Console com o URI de callback da credencial |

> Para Gmail com OAuth2, use o **Connect OAuth (Browser)** na tela de credencial. O Google exige que o URI de callback cadastrado no Console seja idêntico ao exibido no campo **OAuth Callback URI**.

### Outlook / Microsoft 365

| Modo | Configuração necessária |
|------|------------------------|
| **Senha / App Password** | Funciona com contas que não têm MFA, ou com senhas de app |
| **OAuth2** | Registre o app no Azure Active Directory com permissão `IMAP.AccessAsUser.All` |

### IMAP Customizado

Configure manualmente host, porta e modo de segurança (SSL/TLS).

---

## Dicas

- **Sempre use Received After Mode = Step Start** — evita capturar e-mails anteriores ao teste
- **Combine filtros** — quanto mais específico (`from` + `subject`), mais confiável a detecção
- **Marcar como lido** — útil para evitar que o mesmo e-mail seja capturado em execuções seguidas
- **Timeout 0** — executa apenas uma verificação sem aguardar; útil para assertivas em e-mails já recebidos
- **OTP Regex customizado** — use quando o código tiver formato não numérico ou esteja em contexto ambíguo (ex: `código: ([A-Z0-9]{8})`)
