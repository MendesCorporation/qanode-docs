# Dashboard

O **Dashboard** oferece uma visão consolidada do estado dos seus testes através de **widgets** configuráveis — cards de métrica, gráficos e tabelas.

---

## Visão Geral

[Dashboard principal](../../assets/images/dashboard-visao-geral.mp4)

*Imagem: Dashboard com múltiplos widgets: cards de métrica, gráfico de barras e tabela*

---

## Dashboards Múltiplos

O QANode suporta múltiplos dashboards com diferentes visibilidades:

| Tipo | Visibilidade | Descrição |
|------|-------------|-----------|
| **Privado** | Apenas o criador | Dashboard pessoal |
| **Público** | Todos os usuários | Dashboard compartilhado |
| **Por Papel** | Usuários com papéis específicos | Dashboard por equipe/papel |
| **Sistema** | Todos (somente leitura) | Dashboards padrão do QANode |

[Dashboards com diferentes visibilidades](../../assets/images/dashboard-visibilidade-painel.mp4)

---

## Widgets

### Tipos de Widget

| Tipo | Descrição | Uso Típico |
|------|-----------|-----------|
| **Card de Métrica** | Valor numérico único com formatação condicional | Total de execuções, taxa de sucesso |
| **Gauge** | Indicador semicircular com mínimo, máximo e valor atual | Taxa de sucesso, SLA, progresso |
| **Gráfico de Barras** | Barras verticais com uma ou mais séries | Execuções por dia, falhas por projeto |
| **Barras Empilhadas** | Barras verticais com múltiplas séries empilhadas | Comparação acumulada entre categorias |
| **Barras Horizontais** | Barras horizontais com uma ou mais séries | Rankings, comparações categóricas |
| **Gráfico de Linha** | Linhas de tendência | Evolução da taxa de sucesso |
| **Linha Empilhada** | Múltiplas linhas com séries acumuladas | Evolução por status ou equipe |
| **Gráfico de Área** | Área preenchida | Acumulado de execuções |
| **Área Empilhada** | Áreas acumuladas por série | Volume por status ao longo do tempo |
| **Gráfico de Pizza** | Distribuição proporcional | Proporção sucesso/falha |
| **Donut** | Distribuição proporcional com centro livre para total | Distribuição por status com valor central |
| **Funnel** | Etapas em formato de funil | Conversão, qualidade ou etapas de processo |
| **Calendar Heatmap** | Intensidade por dia em calendário | Falhas por dia, execuções por dia |
| **Scatter** | Pontos por eixo X/Y | Duração por quantidade de passos |
| **Bubble** | Scatter com tamanho variável | Duração, passos e severidade |
| **Tabela** | Dados tabulares | Lista de últimas execuções |

### Criando um Widget

O assistente de criação de widget possui 3 etapas:

[Criando um Widget](../../assets/images/dashboard-criando-widget.mp4)

#### Etapa 1: Fonte de Dados

Defina de onde os dados virão:

**Query Builder (Recomendado):**

| Campo | Descrição |
|-------|-----------|
| **Tabela** | Tabela de dados (execuções, projetos, fluxos, etc.) |
| **Colunas** | Campos a selecionar |
| **Filtros** | Condições (igual a, contém, maior que, etc.) |
| **Agrupamento** | Agrupar por campo (com formatação de data) |
| **Agregação** | COUNT, SUM, AVG, MIN, MAX |
| **Ordenação** | Campo e direção (ASC/DESC) |
| **Pivot** | Criar múltiplas séries a partir de um campo |
| **Limite** | Máximo de registros (até 1000) |

**Lógica de Filtros:** Combine múltiplos filtros com operadores **AND** ou **OR**.

**Formatação de Data no Agrupamento:**

| Formato | Resultado |
|---------|-----------|
| Somente Data | `2024-01-15` |
| Somente Hora | `14:30` |
| Data e Hora | `2024-01-15 14:30` |
| Mês/Ano | `2024-01` |
| Ano | `2024` |

**SQL Direto:**

Para consultas mais complexas, use o modo SQL com o editor Monaco. Prefira o Query Builder para widgets simples e use SQL quando precisar de CTEs, joins, agregações condicionais ou cálculos que ficariam difíceis no construtor.

```sql
SELECT
  DATE("startedAt") AS dia,
  status,
  COUNT(*)::int AS total
FROM "Run"
WHERE "isSandbox" = false
  AND "startedAt" >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY dia, status
ORDER BY dia;
```

> O SQL também suporta CTEs (Common Table Expressions) com a cláusula `WITH`. Consultas que modificam dados (INSERT, UPDATE, DELETE, DROP, etc.) continuam bloqueadas por guardrails de leitura.

> O modo SQL requer a permissão `dashboard.sql`.

#### Etapa 2: Visualização

Escolha o tipo de gráfico e mapeie os campos:

| Configuração | Descrição |
|-------------|-----------|
| **Tipo de Gráfico** | Métrica, Gauge, Barras, Linha, Área, Pizza, Donut, Funnel, Heatmap, Scatter, Bubble ou Tabela |
| **Eixo X** | Campo para o eixo horizontal |
| **Eixo Y** | Campo para o eixo vertical (valor numérico) |
| **Série** | Campo para múltiplas séries (quando pivot) |
| **Label** | Campo de rótulo, usado por pizza, donut, funnel, scatter, bubble e tabelas |
| **Tamanho** | Campo numérico usado por Bubble |
| **Legenda** | Exibir/ocultar legenda |
| **Tooltip** | Exibir valores ao passar o mouse |

#### Etapa 3: Aparência

| Configuração | Descrição |
|-------------|-----------|
| **Título** | Nome exibido no widget |
| **Cores** | Cores customizadas por série/categoria |
| **Formatação Condicional** | Regras de cores baseadas em valores |
| **Colunas de Progresso** | Exibe colunas numéricas da tabela como barra de progresso |
| **Limite / Alvo** | Referências visuais para Gauge e indicadores |

**Formatação Condicional** (para cards e tabelas):

| Operador | Exemplo |
|----------|---------|
| `>` | Se valor > 90 → verde |
| `<` | Se valor < 50 → vermelho |
| `=` | Se valor = "failed" → vermelho |
| `contains` | Se contém "error" → amarelo |

### Tabelas com Progresso

Em widgets de tabela, colunas numéricas podem ser exibidas como barra de progresso. Isso é útil para acompanhar:

- avanço de projetos;
- percentual de sucesso;
- cobertura de cenários;
- consumo de SLA;
- progresso de pipelines.

Cada coluna de progresso pode ter cor própria e valores mínimo/máximo. Campos textuais, IDs e datas não são recomendados para progresso.

### Formatação Condicional em Tabelas

As regras de formatação condicional permitem destacar linhas ou células conforme valores da própria consulta.

Exemplos:

| Regra | Resultado |
|-------|-----------|
| `status = failed` | Linha em vermelho |
| `taxa_sucesso < 80` | Indicador de atenção |
| `falhas > 0` | Destacar cenários com falha |

Use formatação condicional para chamar atenção para problemas, mas evite aplicar muitas cores ao mesmo tempo.

---

## Layout do Dashboard

O dashboard usa um **grid de 12 colunas** responsivo:

| Propriedade | Descrição |
|-------------|-----------|
| **Largura** | 1 a 12 colunas |
| **Altura** | 1 a 10 linhas |
| **Posição X** | Coluna de início (0-11) |
| **Posição Y** | Linha de início |

### Reorganizando Widgets

[Reorganizando Widgets](../../assets/images/dashboard-reorganizar-widgets.mp4)

- **Arrastar** um widget para reposicioná-lo
- **Redimensionar** arrastando a borda inferior-direita
- O grid se ajusta automaticamente para evitar sobreposições

---

## Exemplos de Widgets

[Exemplos de Widgets](../../assets/images/dashboard-tipos-de-widgets.mp4)

### Card: Taxa de Sucesso

```
Tabela: runs
Agregação: COUNT(*)
Filtro: status = "success" AND started_at >= hoje - 30 dias
```

### Gráfico de Barras: Execuções por Dia

```
Tabela: runs
Agrupamento: started_at (somente data)
Agregação: COUNT(*)
Pivot: status
Tipo: Barras
```

Resultado: Barras empilhadas com cores diferentes para sucesso/falha por dia.

### Tabela: Últimas Falhas

```
Tabela: runs
Filtro: status = "failed"
Colunas: nome do fluxo, data, duração, erro
Ordenação: started_at DESC
Limite: 10
```

---

## Auto-Refresh

Widgets podem ser configurados para atualizar automaticamente em intervalos definidos, mantendo os dados sempre atualizados.

[Auto-Refresh](../../assets/images/dashboard-auto-refresh.mp4)

---

## Dicas

- **Comece simples** — um card com total de execuções + um gráfico de barras por dia
- Use **formatação condicional** para destacar problemas (taxa abaixo de 80% = vermelho)
- **Dashboards por equipe** — crie dashboards com visibilidade por papel
- Use o **modo pivot** para criar gráficos com múltiplas séries sem SQL
- Use **SQL** apenas quando o Query Builder não expressar bem a consulta
- Em tabelas, use **colunas de progresso** apenas para campos numéricos de negócio
- **Limite os dados** — consultas grandes podem impactar a performance
