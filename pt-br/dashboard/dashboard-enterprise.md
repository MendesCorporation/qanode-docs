# Dashboard

O **Dashboard** oferece uma visão consolidada do estado dos seus testes através de **widgets** configuráveis — cards de métrica, gráficos e tabelas.

---

## Visão Geral

<!-- ![Dashboard principal](../../assets/images/dashboard-principal.png) -->
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

---

## Widgets

### Tipos de Widget

| Tipo | Descrição | Uso Típico |
|------|-----------|-----------|
| **Card de Métrica** | Valor numérico único com formatação condicional | Total de execuções, taxa de sucesso |
| **Gráfico de Barras** | Barras verticais com uma ou mais séries | Execuções por dia, falhas por projeto |
| **Gráfico de Linha** | Linhas de tendência | Evolução da taxa de sucesso |
| **Gráfico de Área** | Área preenchida | Acumulado de execuções |
| **Gráfico de Pizza** | Distribuição proporcional | Proporção sucesso/falha |
| **Tabela** | Dados tabulares | Lista de últimas execuções |

### Criando um Widget

O assistente de criação de widget possui 3 etapas:

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

Para consultas mais complexas, use o modo SQL com o editor Monaco:

```sql
SELECT
  DATE(started_at) as dia,
  status,
  COUNT(*) as total
FROM runs
WHERE started_at >= NOW() - INTERVAL '30 days'
GROUP BY dia, status
ORDER BY dia
```

> O modo SQL requer a permissão `dashboard.sql`.

#### Etapa 2: Visualização

Escolha o tipo de gráfico e mapeie os campos:

| Configuração | Descrição |
|-------------|-----------|
| **Tipo de Gráfico** | Card, Barras, Linha, Área, Pizza, Tabela |
| **Eixo X** | Campo para o eixo horizontal |
| **Eixo Y** | Campo para o eixo vertical (valor numérico) |
| **Série** | Campo para múltiplas séries (quando pivot) |
| **Legenda** | Exibir/ocultar legenda |
| **Tooltip** | Exibir valores ao passar o mouse |

#### Etapa 3: Aparência

| Configuração | Descrição |
|-------------|-----------|
| **Título** | Nome exibido no widget |
| **Cores** | Cores customizadas por série/categoria |
| **Formatação Condicional** | Regras de cores baseadas em valores |

**Formatação Condicional** (para cards e tabelas):

| Operador | Exemplo |
|----------|---------|
| `>` | Se valor > 90 → verde |
| `<` | Se valor < 50 → vermelho |
| `=` | Se valor = "failed" → vermelho |
| `contains` | Se contém "error" → amarelo |

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

- **Arrastar** um widget para reposicioná-lo
- **Redimensionar** arrastando a borda inferior-direita
- O grid se ajusta automaticamente para evitar sobreposições

---

## Exemplos de Widgets

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

---

## Dicas

- **Comece simples** — um card com total de execuções + um gráfico de barras por dia
- Use **formatação condicional** para destacar problemas (taxa abaixo de 80% = vermelho)
- **Dashboards por equipe** — crie dashboards com visibilidade por papel
- Use o **modo pivot** para criar gráficos com múltiplas séries sem SQL
- **Limite os dados** — consultas grandes podem impactar a performance
