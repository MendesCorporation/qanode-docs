# Nó SSH Command

O nó **SSH Command** permite executar comandos em servidores remotos via protocolo SSH. Suporta múltiplos passos (comandos sequenciais), autenticação por senha ou chave privada, e captura da saída.

---

## Visão Geral

[Visão Geral](../../assets/images/nos-ssh-visao-geral.mp4)

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

### Passos SSH

O nó suporta múltiplos passos executados sequencialmente na mesma conexão SSH. Use **+ Adicionar passo** para incluir comandos, envios de arquivo e downloads.

| Tipo de passo | Descrição |
|---------------|-----------|
| **Comando** | Executa um comando no shell remoto |
| **Enviar arquivo** | Envia um `fileRef` para o servidor remoto via SFTP |
| **Baixar arquivo** | Baixa um arquivo remoto via SFTP, com fallback para SCP |

Todos os passos rodam na mesma sessão SSH. Isso significa que comandos como `cd /minha/pasta` afetam os passos seguintes quando eles usam **Pasta atual**.

### Passo Comando

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Rótulo** | `string` | Nome descritivo do passo |
| **Comando** | `string` | Comando a executar (suporta `{{ }}`) |
| **Aguardar Match** | `boolean` | Esperar por texto na saída |
| **String de Match** | `string` | Texto a aguardar |
| **Usar Regex** | `boolean` | Interpretar match como regex |
| **Timeout do Match (ms)** | `number` | Tempo máximo para match |

### Passo Enviar Arquivo

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Rótulo** | `string` | Nome descritivo do passo |
| **Arquivo** | `fileRef` | Arquivo a enviar |
| **Destino** | seleção | **Pasta atual** ou **Caminho remoto** |
| **Caminho remoto** | `string` | Caminho absoluto ou relativo no servidor, quando o destino é caminho remoto |
| **Manter nome do arquivo** | `boolean` | Usa o nome original do arquivo |
| **Nome do arquivo** | `string` | Novo nome no servidor, quando **Manter nome do arquivo** está desligado |
| **Criar pastas remotas** | `boolean` | Cria diretórios faltantes antes do envio |

Com **Destino = Pasta atual**, o QANode envia o arquivo para o diretório atual do shell SSH. Isso permite fazer:

```
Passo 1: cd /opt/minha-app/uploads
Passo 2: Enviar arquivo → Destino: Pasta atual
```

Com **Destino = Caminho remoto**, informe o caminho diretamente. Se terminar com `/`, o arquivo é salvo dentro da pasta informada.

### Passo Baixar Arquivo

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Rótulo** | `string` | Nome descritivo do passo |
| **Origem** | seleção | **Pasta atual** ou **Caminho remoto** |
| **Nome do arquivo** | `string` | Nome do arquivo quando a origem é a pasta atual |
| **Caminho remoto** | `string` | Caminho do arquivo no servidor, quando a origem é caminho remoto |
| **Nome da variável** | `string` | Chave usada em `outputs.files` |
| **Nome do arquivo de saída** | `string` | Nome do arquivo salvo no QANode. Se vazio, usa o nome remoto |

Se a extensão for omitida no caminho remoto, o QANode tenta resolver pelo nome do arquivo na pasta remota. Por exemplo, `/tmp/massas` pode encontrar `/tmp/massas.csv` quando esse arquivo existe no servidor.

O MIME type é inferido automaticamente pelo conteúdo e pelo nome do arquivo.

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
| `exitCode` | `number` | Código de saída (0 = sucesso) |
| `commands` | `array` | Resultado de cada passo individual |
| `files` | `object` | Arquivos baixados, quando existe passo **Baixar arquivo** |
| `fileRef` | `fileRef` | Atalho para o último arquivo baixado |
| `fileInfo` | `object` | Metadados dos arquivos baixados, como nome, MIME type, tamanho, caminho remoto e método de transferência |

### Acessando os Outputs

```
// Saída completa
{{ steps.ssh.outputs.stdout }}

// Código de saída
{{ steps.ssh.outputs.exitCode }}

// Resultado de um passo específico
{{ steps.ssh.outputs.commands[0].stdout }}
{{ steps.ssh.outputs.commands[0].exitCode }}

// Arquivo baixado pelo nome da variável
{{ steps.ssh.outputs.files.relatorio }}

// Atalho para o último arquivo baixado
{{ steps.ssh.outputs.fileRef }}
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

### Enviar e baixar arquivo na pasta atual

```
Passo 1:
  Rótulo: Entrar na pasta
  Comando: cd /root/testes

Passo 2:
  Rótulo: Enviar massa
  Tipo: Enviar arquivo
  Arquivo: {{ steps["file-generate"].outputs.fileRef }}
  Destino: Pasta atual
  Manter nome do arquivo: true

Passo 3:
  Rótulo: Baixar resultado
  Tipo: Baixar arquivo
  Origem: Pasta atual
  Nome do arquivo: resultado.csv
  Nome da variável: resultado
```

Depois do download:

```
{{ steps.ssh.outputs.files.resultado }}
{{ steps.ssh.outputs.fileInfo.resultado.name }}
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
- Use **Pasta atual** quando um comando anterior já posicionou o shell no diretório correto
- Use **Caminho remoto** quando quiser ser explícito e independente do `cd` anterior
- Para baixar arquivos, prefira um **Nome da variável** claro, como `relatorio`, `massa` ou `evidencia`
- Comandos com `{{ }}` são avaliados antes da execução
