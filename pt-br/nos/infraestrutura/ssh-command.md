# Nó SSH Command

O nó **SSH Command** permite executar comandos em servidores remotos via protocolo SSH. Suporta múltiplos passos (comandos sequenciais), autenticação por senha ou chave privada, e captura da saída.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `ssh-command` |
| **Categoria** | Infraestrutura |
| **Cor** | 🔵 Azul Claro (#0ea5e9) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Conexão

### Usando Credenciais Salvas (Recomendado)

Selecione uma credencial do tipo **SSH** no campo **Credencial**.

### Configuração Manual

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Host** | `string` | Endereço do servidor |
| **Porta** | `number` | Porta SSH (padrão: 22) |
| **Usuário** | `string` | Nome de usuário |
| **Senha** | `string` | Senha (autenticação por senha) |
| **Chave Privada** | `string` | Chave privada SSH (autenticação por chave) |
| **Passphrase** | `string` | Frase-senha da chave (se protegida) |

### Métodos de Autenticação

| Método | Campos Necessários |
|--------|-------------------|
| **Senha** | Usuário + Senha |
| **Chave Privada** | Usuário + Chave Privada + Passphrase (opcional) |

---

## Configuração

### Timeout Global

| Campo | Tipo | Padrão | Descrição |
|-------|------|--------|-----------|
| **Timeout (ms)** | `number` | 30000 | Tempo máximo para a execução total |

### Passos (SSH Steps)

O nó suporta múltiplos passos executados sequencialmente na mesma conexão SSH:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Label** | `string` | Nome descritivo do passo |
| **Comando** | `string` | Comando a executar (suporta `{{ }}`) |
| **Aguardar Match** | `boolean` | Esperar por texto na saída |
| **String de Match** | `string` | Texto a aguardar |
| **Usar Regex** | `boolean` | Interpretar match como regex |
| **Timeout do Match (ms)** | `number` | Tempo máximo para match |

### Aguardar Match

O recurso de **Aguardar Match** permite que o passo espere até que um texto específico apareça na saída do comando. Útil para:
- Aguardar serviços iniciarem
- Esperar prompts interativos
- Confirmar que uma operação foi concluída

**Exemplo:**
```
Comando: systemctl restart nginx
Aguardar Match: true
String de Match: "Active: active (running)"
Timeout: 10000
```

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `stdout` | `string` | Saída padrão completa |
| `stderr` | `string` | Saída de erro |
| `exitCode` | `number` | Código de saída (0 = sucesso) |
| `steps` | `array` | Resultado de cada passo individual |

### Acessando os Outputs

```
// Saída completa
{{ steps.ssh.outputs.stdout }}

// Código de saída
{{ steps.ssh.outputs.exitCode }}

// Resultado de um passo específico
{{ steps.ssh.outputs.steps[0].stdout }}
{{ steps.ssh.outputs.steps[0].exitCode }}
```

---

## Exemplos Práticos

### Verificar status de serviço

```
Passo 1:
  Label: Verificar Nginx
  Comando: systemctl status nginx
```

### Deploy de aplicação

```
Passo 1:
  Label: Ir para diretório
  Comando: cd /opt/minha-app

Passo 2:
  Label: Pull do repositório
  Comando: git pull origin main

Passo 3:
  Label: Instalar dependências
  Comando: npm install --production

Passo 4:
  Label: Reiniciar serviço
  Comando: pm2 restart minha-app
  Aguardar Match: true
  String de Match: "online"
```

### Coletar informações do servidor

```
Passo 1:
  Label: Uso de disco
  Comando: df -h /

Passo 2:
  Label: Memória
  Comando: free -m

Passo 3:
  Label: Processos
  Comando: ps aux --sort=-%mem | head -10
```

---

## Fluxo com Validação

```
[SSH Command: "deploy"]
    │
    ▼
[If: {{ steps.ssh.outputs.exitCode }} === 0]
    │ true → [HTTP Request: GET /health-check]
    │            │
    │            ▼
    │         [Assert: status === 200]
    │
    │ false → [Log: "Deploy falhou: {{ steps.ssh.outputs.stderr }}"]
              → [Stop and Fail]
```

---

## Dicas

- **Use credenciais salvas** para evitar expor senhas/chaves nos fluxos
- **Verifique o exitCode** — `0` indica sucesso, outros valores indicam erro
- Use **expressões** nos comandos: `echo "Deploy em {{ variables.ENVIRONMENT }}"`
- O **aguardar match** é essencial para comandos que demoram (restarts, builds)
- Todos os passos rodam na **mesma sessão SSH** — o diretório de trabalho é mantido entre passos
- Comandos com `{{ }}` são avaliados antes da execução
