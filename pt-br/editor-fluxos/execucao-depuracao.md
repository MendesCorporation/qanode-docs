# Execução e Depuração

Aprenda como executar seus fluxos, interpretar resultados e diagnosticar falhas.

---

## Executando um Fluxo

### Execução Completa

Para executar todo o fluxo:

1. Certifique-se de que o fluxo está salvo
2. Clique no botão **Executar** (▶️) na barra superior
3. Aguarde a conclusão — os nós exibirão indicadores de status em tempo real

[Fluxo em execução](../../assets/images/executando.mp4) 
*Imagem: Canvas mostrando nós com indicadores de status durante a execução*

### Status de Execução

| Status | Indicador | Descrição |
|--------|-----------|-----------|
| **Executando** | 🔵 Azul/Pulsante | O nó está sendo processado |
| **Sucesso** | 🟢 Verde | O nó foi executado com sucesso |
| **Falha** | 🔴 Vermelho | O nó encontrou um erro |
| **Pulado** | ⚪ Cinza | O nó não foi executado (branch inativo) |

---

## Analisando Resultados

Após a execução, clique em qualquer nó para ver seus resultados no painel de propriedades:

### Aba de Resultados

| Seção | Descrição |
|-------|-----------|
| **Status** | Sucesso ou falha, com duração |
| **Logs** | Mensagens registradas durante a execução do nó |
| **Outputs** | Dados produzidos pelo nó (JSON navegável) |
| **Erro** | Mensagem de erro detalhada (quando aplicável) |
| **Screenshots** | Capturas de tela (nós web com evidência ativada) |

### Outputs

Os outputs de cada nó ficam acessíveis para nós seguintes via expressões. Por exemplo, após executar um **HTTP Request**, os outputs serão:

```json
{
  "status": 200,
  "body": "...",
  "json": {
    "id": 1,
    "name": "João",
    "email": "joao@exemplo.com"
  }
}
```

Você pode então acessar esses dados em nós seguintes:
```
{{ steps["http-request"].outputs.json.name }}  →  "João"
{{ steps["http-request"].outputs.status }}     →  200
```

### Screenshots (Evidências)

Para nós web (Web Flow e Smart Locators), screenshots capturados aparecem como miniaturas clicáveis. Clique para ver em tamanho completo.

---

## Depurando Falhas

### Identificando o Nó com Problema

1. Procure nós com borda **vermelha** (🔴) no canvas
2. Clique no nó com falha
3. Verifique a **mensagem de erro** e os **logs**

### Mensagens de Erro Comuns

#### Web Flow / Smart Locators

| Erro | Causa | Solução |
|------|-------|---------|
| `Timeout waiting for selector` | Elemento não encontrado na página | Verifique o seletor/localizador; aumente o timeout |
| `Element not visible` | Elemento existe mas não está visível | Adicione um passo de `wait` ou `scroll` antes |
| `Navigation timeout` | Página não carregou a tempo | Verifique a URL e a conectividade |
| `Element is not attached to DOM` | Elemento removido antes da ação | Adicione um `wait` para estabilidade |

#### HTTP Request

| Erro | Causa | Solução |
|------|-------|---------|
| `ECONNREFUSED` | Servidor não está acessível | Verifique a URL e se o servidor está rodando |
| `401 Unauthorized` | Autenticação inválida | Verifique token/credenciais |
| `ETIMEOUT` | Requisição excedeu o tempo limite | Aumente o timeout ou verifique o servidor |

#### Banco de Dados

| Erro | Causa | Solução |
|------|-------|---------|
| `Connection refused` | Banco não acessível | Verifique host, porta e firewall |
| `Authentication failed` | Credenciais inválidas | Verifique usuário e senha |
| `Relation does not exist` | Tabela não encontrada | Verifique o nome da tabela e o banco |

#### SSH

| Erro | Causa | Solução |
|------|-------|---------|
| `Authentication failed` | Credenciais SSH inválidas | Verifique usuário, senha ou chave privada |
| `Connection timeout` | Host não acessível | Verifique host, porta e rede |

### Usando Logs para Diagnóstico

Os **logs** de cada nó fornecem detalhes sobre cada passo executado. Para nós web, os logs mostram:

```
Navigated to: https://exemplo.com/login
Filled: "usuario@exemplo.com" on [getByLabel("E-mail")]
Clicked: [getByRole("button", { name: "Entrar" })]
Assert passed: textContains "Bem-vindo" — true
```

Se um passo falhou, o log mostrará exatamente qual passo e por quê:

```
Navigated to: https://exemplo.com/login
Filled: "usuario@exemplo.com" on [getByLabel("E-mail")]
ERROR: Click failed after 3 attempts: [getByRole("button", { name: "Login" })] — element not found
```

---

## Dicas de Depuração

### 1. Desative o Modo Headless

Para testes web, desative o **modo headless** no nó para ver o navegador em ação:
- No nó Web Flow ou Smart Locators, desmarque **Headless**
- Execute o fluxo e observe o navegador

### 2. Adicione Passos de Wait

Se elementos aparecem com atraso, adicione passos de **wait** antes de interagir:
- `wait` com modo `visible` aguarda o elemento aparecer
- `wait` com modo `networkIdle` aguarda todas as requisições de rede terminarem

### 3. Capture Screenshots para Diagnóstico

Ative screenshots no modo **antes** para ver o estado da página antes de cada ação. Isso ajuda a identificar se a página estava no estado esperado.

### 4. Verifique Expressões

Se um nó falha com valor inesperado, adicione um nó **Log** antes dele para inspecionar os valores:

```
Valor do token: {{ steps.login.outputs.json.token }}
Status: {{ steps["http-request"].outputs.status }}
```

### 5. Use o Toggle "Continuar em Falha"

Para diagnosticar múltiplas falhas de uma vez, ative **Continuar em Falha** nos nós de verificação. Isso permite que o fluxo continue e você veja todas as falhas em uma única execução.

---

## Abrindo um Defeito a partir de uma Execução — Enterprise

Quando uma execução termina com **falha**, o botão **Abrir Defeito** fica disponível no detalhe da execução. Ao clicar, um modal abre para preencher as informações do defeito — título, descrição, severidade, prioridade e demais campos definidos pelo workflow.

O defeito criado fica automaticamente vinculado à execução, ao cenário e ao step que originou a falha.

> **Permissão necessária:** `bug.create`

### Defeitos relacionados

O detalhe da execução também exibe a seção **Defeitos Relacionados** com a lista de todos os defeitos vinculados àquela execução — bug key, título, status e severidade — com link direto para cada defeito.

---

## Relatório de Execução

Após cada execução, o QANode gera automaticamente um **relatório PDF** com:

- Resumo da execução (status, duração, data)
- Detalhes de cada passo (status, logs, outputs, duração)
- Screenshots capturados
- Erros encontrados

O relatório fica disponível na lista de execuções do projeto e pode ser baixado ou enviado por e-mail.

---

## Próximos Passos

- [Referência de Nós](../nos/controle-fluxo/if.md) — Detalhes completos de cada tipo de nó
- [Expressões](../expressoes/expressoes.md) — Domine o sistema de dados dinâmicos
