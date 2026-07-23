# Execução e Depuração

Aprenda como executar seus fluxos, interpretar resultados e diagnosticar falhas.

---

## Executando um Fluxo

### Execução Completa

Para executar todo o fluxo:

1. Certifique-se de que o fluxo está salvo
2. Clique no botão **Executar** (▶️) na barra superior
3. Aguarde a conclusão — os nós exibirão indicadores de status em tempo real

[Fluxo em execução](../../assets/images/execucao-completa-fluxos.mp4)
*Imagem: Canvas mostrando nós com indicadores de status durante a execução*

### Execução Visual de Fluxos Web — Enterprise

Ao iniciar uma execução completa pelo botão **Executar** do canvas, desligar **Headless** em pelo menos um nó **Smart Web Flow**, **Web Flow** ou **Smart Locators** ativa o modo visual daquela execução.

O comportamento depende de onde o QANode está sendo executado:

| Ambiente | Comportamento |
|----------|---------------|
| **Servidor com Live View** | Um único painel acompanha os nós web da execução. O nó com **Headless** desligado funciona como gatilho; depois disso, os demais nós web da mesma execução também são exibidos no painel, mesmo que mantenham **Headless** ligado. |
| **Desktop ou ambiente com janela gráfica disponível** | A configuração continua sendo aplicada por nó. Nós com **Headless** desligado podem abrir uma janela visível; nós com **Headless** ligado executam sem janela. |

O Live View pertence à execução iniciada no canvas, e não a um único nó. Por isso:

- o painel permanece aberto até a execução terminar, falhar, ser cancelada ou o usuário fechá-lo;
- quando o fluxo passa por nós que não usam navegador, como **HTTP Request**, **IF**, banco de dados ou **Wait**, nenhuma nova imagem web é produzida e a última tela pode permanecer visível;
- se outro nó web for executado depois, o mesmo painel volta a acompanhar o navegador, sem fechar e abrir uma nova janela;
- manter o painel aberto não significa necessariamente que o navegador anterior continua em execução: o ciclo de vida do navegador ainda depende da estratégia de sessão configurada no nó.

Esse modo visual é aceito somente para uma execução interativa iniciada por um usuário no canvas. Execuções de **suítes**, **agendamentos**, **CI/CD**, **tokens de integração**, **API** ou outros processos externos não abrem janela nem Live View automaticamente.

Ao usar **Executar nó** no painel de propriedades, a visualização continua limitada ao nó selecionado.

### Status de Execução

| Status | Indicador | Descrição |
|--------|-----------|-----------|
| **Executando** | 🔵 Azul/Pulsante | O nó está sendo processado |
| **Sucesso** | 🟢 Verde | O nó foi executado com sucesso |
| **Falha** | 🔴 Vermelho | O nó encontrou um erro |
| **Cancelado** | ⚪ Cinza | A execução foi interrompida manualmente |
| **Pulado** | ⚪ Cinza | O nó não foi executado (branch inativo ou nó marcado como ignorado) |

---

## Cancelando Execuções

Execuções em andamento podem ser canceladas por usuários com permissão `run.cancel`.

Onde cancelar:

- **Editor do cenário**: durante a execução, o botão de executar muda para um botão de parada.
- **Lista de execuções**: use o menu de ações da execução em andamento.
- **Execução de suíte**: cancelar a execução principal também cancela os cenários pendentes ou em execução daquela suíte.

Ao cancelar, o status final fica como **Cancelado**. Evidências e logs já gerados até aquele ponto permanecem disponíveis para consulta.

O cancelamento acontece em pontos seguros da execução. Nós web, Smart Web, mobile, API, banco de dados, SSH, carga e JavaScript customizado verificam o pedido de cancelamento durante o processamento. Em nós com navegador ou sessão mobile, o QANode também tenta encerrar recursos abertos para evitar sessões presas.

> Uma ação externa muito longa pode terminar o trecho atual antes de reconhecer o cancelamento. Nesses casos, o QANode cancela assim que a execução volta para um ponto seguro.

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
| **Possível causa** | Dica amigável exibida em alguns passos com falha, com sugestão de correção |
| **Arquivos e Screenshots** | Artefatos gerados, downloads, uploads capturados e evidências |

[Analisando Resultados](../../assets/images/analise-resultados-editor-fluxos.mp4)

### Outputs

Os outputs de cada nó ficam acessíveis para nós seguintes via expressões. Por exemplo, após executar um **HTTP Request**, os outputs serão:

```json
{
  "status": 200,
  "body": {
    "id": 1,
    "name": "João",
    "email": "joao@exemplo.com"
  }
}
```

Você pode então acessar esses dados em nós seguintes:
```
{{ steps["http-request"].outputs.body.name }}  →  "João"
{{ steps["http-request"].outputs.status }}     →  200
```

Quando o output é um arquivo, o valor principal é um `fileRef`:

```
{{ steps["http-request"].outputs.fileRef }}
{{ steps["file-generate"].outputs.fileRef }}
```

No painel de variáveis, arquivos mostram nome, MIME type e tamanho em detalhes. Campos internos como caminho no storage ficam ocultos da visualização principal.

### Screenshots (Evidências)

Para nós web (Smart Web Flow, Web Flow e Smart Locators), screenshots capturados aparecem como miniaturas clicáveis. Clique para ver em tamanho completo.

### Arquivos

Arquivos gerados ou capturados durante a execução aparecem na seção **Arquivos** do detalhe da execução. Ao baixar, o navegador usa o nome real do arquivo quando disponível, em vez de um identificador interno da execução.

Arquivos de teste isolado criados no editor são temporários e podem ser limpos automaticamente quando o usuário sai do fluxo ou por rotina de limpeza.

---

## Depurando Falhas

### Identificando o Nó com Problema

1. Procure nós com borda **vermelha** (🔴) no canvas
2. Clique no nó com falha
3. Verifique a **mensagem de erro** e os **logs**

### Dicas Amigáveis de Falha

Quando um passo falha por um erro conhecido, o detalhe da execução pode mostrar uma seção **Possível causa** com uma sugestão objetiva. Ela aparece apenas em passos com falha e o QANode mostra uma dica por vez para evitar mensagens conflitantes.

As dicas cobrem problemas comuns como sessão expirada, credenciais, expressões quebradas, arquivos ausentes, alvo ambíguo, campo oculto, desabilitado ou somente leitura, clique bloqueado, timeout, assert que não bateu e efeito esperado que não apareceu depois da ação.

Use a dica como ponto de partida e confirme nas evidências, logs e screenshots antes de alterar o fluxo.

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
- No nó Smart Web Flow, Web Flow ou Smart Locators, desmarque **Headless**
- Ao usar **Executar nó**, a visualização mostra somente o nó selecionado
- Ao executar o fluxo completo pelo canvas em um servidor, o **Live View** acompanha a execução web completa no mesmo painel
- Suítes, agendamentos, CI/CD e chamadas externas continuam executando sem abrir a visualização interativa

### 2. Adicione Passos de Wait

Se elementos aparecem com atraso, adicione passos de **wait** antes de interagir:
- `wait` com modo `visible` aguarda o elemento aparecer
- `wait` com modo `networkIdle` aguarda todas as requisições de rede terminarem

### 3. Capture Screenshots para Diagnóstico

Ative screenshots no modo **antes** para ver o estado da página antes de cada ação. Isso ajuda a identificar se a página estava no estado esperado.

### 4. Verifique Expressões

Se um nó falha com valor inesperado, adicione um nó **Log** antes dele para inspecionar os valores:

```
Valor do token: {{ steps.login.outputs.body.token }}
Status: {{ steps["http-request"].outputs.status }}
```

### 5. Use o Toggle "Continuar em Falha"

Para diagnosticar múltiplas falhas de uma vez, ative **Continuar em Falha** nos nós de verificação. Isso permite que o fluxo continue e você veja todas as falhas em uma única execução.

---

## Abrindo um Defeito a partir de uma Execução — Enterprise

Quando uma execução termina com **falha**, o botão **Abrir Defeito** fica disponível no detalhe da execução. Ao clicar, um modal abre para preencher as informações do defeito — título, descrição, severidade, prioridade e demais campos definidos pelo workflow.

O defeito criado fica automaticamente vinculado à execução, ao cenário e ao step que originou a falha.

> **Permissão necessária:** `bug.create`

[Abrindo um Defeito](../../assets/images/abrir-defeito-editor-fluxos.mp4)

### Defeitos relacionados

O detalhe da execução também exibe a seção **Defeitos Relacionados** com a lista de todos os defeitos vinculados àquela execução — bug key, título, status e severidade — com link direto para cada defeito.

---

## Relatório de Execução

Após cada execução, o QANode disponibiliza um **relatório PDF** com:

- Resumo da execução (status, duração, data)
- Detalhes de cada passo (status, logs, outputs, duração)
- Screenshots capturados
- Erros encontrados

O relatório fica disponível na lista de execuções do projeto e pode ser baixado ou enviado por e-mail. Dependendo do template ativo, o PDF pode ser gerado automaticamente ao fim da execução ou apenas quando alguém solicita o download.

[Relatório de Execução](../../assets/images/relatorio-editor-fluxos.mp4)

---

## Próximos Passos

- [Referência de Nós](../nos/controle-fluxo/if.md) — Detalhes completos de cada tipo de nó
- [Expressões](../expressoes/expressoes.md) — Domine o sistema de dados dinâmicos
