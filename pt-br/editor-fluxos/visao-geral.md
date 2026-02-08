# Editor de Fluxos — Visão Geral

O **Editor de Fluxos** é o coração do QANode. É nele que você cria, configura e executa seus cenários de teste de forma visual.

---

## Interface do Editor

O editor é dividido em quatro áreas principais:

<!-- ![Visão geral do editor de fluxos](../../assets/images/editor-visao-geral.png) -->
*Imagem: Editor de fluxos mostrando as quatro áreas: barra superior, paleta de nós, canvas e painel de propriedades*

### 1. Barra Superior

A barra superior contém:

| Elemento | Descrição |
|----------|-----------|
| **Botão Voltar** | Retorna ao projeto (exibe aviso se houver alterações não salvas) |
| **Nome do Fluxo** | Campo editável com o nome do cenário |
| **Botão Salvar** | Salva o fluxo atual (Ctrl+S) |
| **Botão Executar** | Executa o fluxo e exibe os resultados |
| **Menu de Opções** | Exportar, importar, duplicar e outras opções |

### 2. Paleta de Nós (Lado Esquerdo)

A paleta exibe todos os nós disponíveis organizados por categoria:

- **Controle de Fluxo** — If, Switch, Loop, Merge
- **Web** — Web Flow, Smart Locators
- **API** — HTTP Request
- **Banco de Dados** — PostgreSQL, MySQL, MariaDB, Oracle, MongoDB
- **Infraestrutura** — SSH Command
- **Utilitários** — Set Variable, Log, Wait, Stop and Fail, Custom JavaScript
- **Nós Customizados** — Nós de provedores registrados (se houver)

Para adicionar um nó, **arraste-o** da paleta para o canvas.

### 3. Canvas (Área Central)

O canvas é a área de trabalho onde você posiciona e conecta os nós. Funcionalidades:

| Ação | Como Fazer |
|------|-----------|
| **Mover o canvas** | Clique e arraste no fundo (ou use o scroll do mouse) |
| **Zoom** | Scroll do mouse ou pinch no trackpad |
| **Selecionar nó** | Clique no nó |
| **Mover nó** | Arraste o nó |
| **Selecionar múltiplos** | Segure Shift e clique, ou arraste uma caixa de seleção |
| **Deletar nó** | Selecione e pressione Delete ou Backspace |
| **Conectar nós** | Arraste de um handle de saída (●) para um handle de entrada (●) |
| **Remover conexão** | Clique na conexão e pressione Delete |
| **Colar fluxo** | Ctrl+V com JSON de fluxo copiado (ex: do Gravador Chrome) |

### 4. Painel de Propriedades (Lado Direito)

Ao selecionar um nó, o painel de propriedades aparece à direita com:

- **Executar**: Execução do node em modo isolado
- **Variaveis**: Abre as propriedades de nodes anteriores, variaveis locais e variaveis globais
- **Configuração**: Campos específicos do tipo de nó
- **Outputs**: Schema de saída do nó (após execução, mostra os dados reais)
- **Continuar em Falha**: Toggle para continuar o fluxo mesmo se o nó falhar

---

## Fluxo de Trabalho Típico

1. **Crie o cenário** a partir de um projeto
2. **Arraste os nós** da paleta para o canvas
3. **Conecte os nós** arrastando entre handles
4. **Configure cada nó** usando o painel de propriedades
5. **Salve** o fluxo (Ctrl+S)
6. **Execute** e verifique os resultados

---

## Ordem de Execução

O QANode executa os nós em **ordem topológica**, determinada pelas conexões:

```
[A] → [B] → [D]
         ↘
[C] ──────→ [E]
```

Neste exemplo:
- `A` executa primeiro
- `B` e `C` podem executar em seguida (após `A`)
- `D` executa após `B`
- `E` executa após `B` e `C`

> **Nota:** Se um nó tem múltiplas entradas, ele só executa quando todos os nós anteriores forem concluídos.

---

## Indicadores Visuais

Após uma execução, os nós exibem indicadores de status:

| Indicador | Significado |
|-----------|-------------|
| 🟢 Borda verde | Nó executado com sucesso |
| 🔴 Borda vermelha | Nó falhou |
| ⚪ Borda cinza | Nó foi pulado (branch não ativado) |
| 🔵 Borda azul | Nó em execução |

---

## Atalhos de Teclado

| Atalho | Ação |
|--------|------|
| `Ctrl+S` | Salvar fluxo |
| `Delete` / `Backspace` | Deletar nó ou conexão selecionada |
| `Ctrl+V` | Colar nó (JSON do Gravador Chrome) |
| `Ctrl+Z` | Desfazer |
| `Ctrl+Shift+Z` | Refazer |

---

## Alterações Não Salvas

O editor detecta automaticamente quando há alterações não salvas. Ao tentar sair da página (pelo botão voltar, navegação ou fechamento da aba), um aviso será exibido:

> **"Sair sem salvar?"**
> Você tem alterações não salvas. Se sair, suas alterações serão perdidas.

Isso protege contra perda acidental de trabalho. O aviso só aparece quando há alterações reais — não ao abrir um fluxo sem modificá-lo.

---

## Próximos Passos

- [Trabalhando com Nós](trabalhando-com-nos.md) — Detalhes sobre adicionar, conectar e configurar nós
- [Execução e Depuração](execucao-depuracao.md) — Como executar fluxos e diagnosticar falhas
