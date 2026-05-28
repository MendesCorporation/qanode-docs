# Editor de Fluxos — Visão Geral

O **Editor de Fluxos** é o coração do QANode. É nele que você cria, configura e executa seus cenários de teste de forma visual.

---

## Interface do Editor

O editor é dividido em quatro áreas principais:

[Visão geral do editor de fluxos](../../assets/images/interface-editor-fluxos.mp4)

*Imagem: Editor de fluxos mostrando as quatro áreas: barra superior, paleta de nós, canvas e painel de propriedades*

### 1. Barra Superior

A barra superior contém:

| Elemento | Descrição |
|----------|-----------|
| **Botão Voltar** | Retorna ao projeto (exibe aviso se houver alterações não salvas) |
| **Nome do Fluxo** | Campo editável com o nome do cenário |
| **Botão Salvar** | Salva o fluxo atual (Ctrl+S) |
| **Botão Executar** | Executa o fluxo e exibe os resultados |
| **Histórico de Versões** | Abre snapshots salvos do cenário para consulta e restauração (Enterprise) |
| **Menu de Opções** | Exportar, importar, duplicar e outras opções |

### 2. Paleta de Nós (Lado Esquerdo)

A paleta exibe todos os nós disponíveis organizados por categoria:

- **Controle de Fluxo** — If, Switch, Loop, Merge
- **Web** — Smart Web Flow, Web Flow, Smart Locators
- **API** — HTTP Request
- **Banco de Dados** — PostgreSQL, MySQL, MariaDB, SQL Server, Oracle, MongoDB
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
| **Adicionar anotação** | Clique com botão direito no canvas ou pressione Shift+A |
| **Copiar/colar nó ou anotação** | Selecione o item e use Ctrl+C / Ctrl+V |

### 4. Painel de Propriedades (Lado Direito)

Ao selecionar um nó, o painel de propriedades aparece à direita com:

- **Executar**: Execução do node em modo isolado
- **Variáveis**: Abre as propriedades de nodes anteriores, variáveis locais, variáveis globais e, quando aplicável, variáveis do projeto
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

[Fluxo de Trabalho Típico](../../assets/images/fluxo-trabalho-editor-fluxos.mp4)

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

[Indicadores Visuais](../../assets/images/indicadores-visuais-editor-fluxos.mp4)

---

## Atalhos de Teclado

| Atalho | Ação |
|--------|------|
| `Ctrl+S` | Salvar fluxo |
| `Delete` / `Backspace` | Deletar nó ou conexão selecionada |
| `Ctrl+C` | Copiar nó |
| `Ctrl+V` | Colar nó, anotação ou JSON do Gravador Chrome |
| `Ctrl+Z` | Desfazer |
| `Ctrl+Shift+Z` | Refazer |
| `Shift+A` | Criar anotação no canvas |

### Anotações no Canvas

As **anotações** ajudam a documentar partes do fluxo diretamente no canvas. Use-as para explicar pré-condições, regras de negócio, agrupamentos lógicos, pendências ou observações importantes para o time.

As anotações suportam: 

- texto livre;
- formatação simples em estilo Markdown;
- cores;
- redimensionamento;
- arrastar e posicionar no canvas;
- copiar, colar, duplicar e excluir.

Você pode criar uma anotação pelo menu de contexto do canvas ou pelo atalho `Shift+A`.

[Anotações no Canvas](../../assets/images/anotacoes-editor-fluxos.mp4) 

---

## Alterações Não Salvas

O editor detecta automaticamente quando há alterações não salvas. Ao tentar sair da página (pelo botão voltar, navegação ou fechamento da aba), um aviso será exibido:

> **"Sair sem salvar?"**
> Você tem alterações não salvas. Se sair, suas alterações serão perdidas.

Isso protege contra perda acidental de trabalho. O aviso só aparece quando há alterações reais — não ao abrir um fluxo sem modificá-lo.

---

## Histórico de Versões — Enterprise

O QANode Enterprise pode manter snapshots do cenário ao salvar alterações relevantes no fluxo. O histórico ajuda a auditar mudanças, comparar versões e voltar para uma versão anterior quando necessário.

Para acessar:

1. Abra o cenário no editor.
2. Clique no ícone de **Histórico de Versões** na barra superior.
3. Selecione uma versão para visualizar.

Ao abrir uma versão antiga, o cenário é exibido em modo de consulta. Você pode navegar pelo canvas, selecionar nós e conferir parâmetros, mas não edita diretamente o snapshot.

Para voltar a uma versão anterior, use **Restaurar esta versão**. O QANode aplica aquele snapshot como a versão atual do cenário e preserva a rastreabilidade da operação.

> O histórico de versões fica disponível apenas no Enterprise. A quantidade de versões mantidas é definida pelo Super Admin em **Configurações → Geral**.

---

## Próximos Passos

- [Trabalhando com Nós](trabalhando-com-nos.md) — Detalhes sobre adicionar, conectar e configurar nós
- [Execução e Depuração](execucao-depuracao.md) — Como executar fluxos e diagnosticar falhas
