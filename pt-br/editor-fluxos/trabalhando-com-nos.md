# Trabalhando com Nós

Este guia detalha como adicionar, conectar, configurar e gerenciar nós no editor de fluxos.

---

## Adicionando Nós

### Arrastar da Paleta

A forma principal de adicionar nós é arrastando da **paleta de nós** (lado esquerdo) para o **canvas**:

1. Localize a categoria desejada na paleta
2. Clique e arraste o nó para o canvas
3. Solte na posição desejada

<!-- ![Arrastando um nó da paleta](../../assets/images/arrastar-no.png) -->
*Imagem: Nó sendo arrastado da paleta de nós para o canvas*

### Colar JSON (Gravador Chrome)

Você também pode colar nós copiados do **Gravador Chrome** (extensão):

1. No Gravador Chrome, clique em **Copiar JSON**
2. No editor de fluxos, pressione **Ctrl+V**
3. O nó Web Flow será adicionado com todos os passos gravados

---

## Conectando Nós

### Criando Conexões

Para conectar dois nós:

1. Passe o mouse sobre o **handle de saída** (●) do nó de origem
2. Clique e arraste até o **handle de entrada** (●) do nó de destino
3. Solte para criar a conexão

<!-- ![Conectando nós](../../assets/images/conectar-nos.png) -->
*Imagem: Linha sendo arrastada de um handle de saída para um handle de entrada*

### Handles de Entrada e Saída

| Tipo | Posição | Descrição |
|------|---------|-----------|
| **Entrada** (in) | Topo do nó | Recebe dados de nós anteriores |
| **Saída** (out) | Base do nó | Envia dados para nós seguintes |

Alguns nós possuem **múltiplos handles de saída**:

- **If** → `true` e `false`
- **Switch** → Um handle por case + `default`
- **Loop** → `loop` (corpo do loop) e `done` (saída do loop)

### Removendo Conexões

Para remover uma conexão:
1. Clique na linha de conexão para selecioná-la
2. Pressione **Delete** ou **Backspace**

---

## Configurando Nós

### Painel de Propriedades

Ao clicar em um nó, o painel de propriedades abre à direita. Cada tipo de nó tem campos específicos, mas todos compartilham:

| Campo | Descrição |
|-------|-----------|
| **Label** | Nome do nó (exibido no canvas) |
| **Continuar em Falha** | Se ativado, o fluxo continua mesmo se este nó falhar |

### Usando Expressões nos Campos

A maioria dos campos aceita **expressões** com a sintaxe `{{ }}`:

```
{{ steps['Nome do Nó'].outputs.propriedade }}
{{ variables.minhaVariavel }}
```

Isso permite que nós usem dados produzidos por nós anteriores. Por exemplo:

- **URL de navegação**: `{{ variables.BASE_URL }}/login`
- **Texto para preencher**: `{{ steps['Nome do Nó'].outputs.result.email }}`
- **SQL**: `SELECT * FROM users WHERE email = '{{ steps.extract.outputs.extracts.email }}'`

> Para mais detalhes, veja [Expressões](../expressoes/expressoes.md).

---

## Nós com Múltiplos Passos

Alguns nós suportam **múltiplos passos** internos, tornando-os mais poderosos:

### Web Flow e Smart Locators

Esses nós permitem adicionar vários passos de automação web dentro de um único nó:

1. No painel de propriedades, clique em **+ Adicionar Passo**
2. Selecione o tipo de ação (navigate, click, fill, assert, etc.)
3. Configure os parâmetros do passo
4. Repita para adicionar mais passos

Os passos são executados **sequencialmente**, na ordem em que aparecem na lista.

Você pode:
- **Reordenar** passos arrastando
- **Expandir/Recolher** passos para ver detalhes
- **Remover** passos clicando no ícone de lixeira
- **Configurar evidências** (screenshots) individualmente por passo

### SSH Command

O nó SSH também suporta múltiplos passos (comandos):

1. Cada passo é um comando a ser executado
2. Passos são executados sequencialmente na mesma conexão SSH
3. Opcionalmente, aguarde por um texto específico na saída (match string)

---

## Nós de Controle de Fluxo

### Criando Desvios Condicionais

Para criar um desvio condicional:

1. Adicione um nó **If** ao canvas
2. Configure a condição no painel de propriedades
3. Conecte a saída **true** ao caminho que deve ser executado quando a condição for verdadeira
4. Conecte a saída **false** ao caminho alternativo

**Exemplo:**

```
[HTTP Request] → [If status === 200]
                       │ true → [Log "Sucesso"]
                       │ false → [Log "Erro"] → [Stop and Fail]
```

### Condição Simples vs Visual Builder

O nó **If** oferece dois modos de configuração:

**Modo Simples:**
```javascript
{{ steps['HTTP Request'].outputs.status }} === 200
```

**Modo Visual Builder:**
- Campo: `steps['HTTP Request'].outputs.status`
- Operador: `===`
- Valor: `200`

O modo visual builder é mais amigável e não requer conhecimento de JavaScript.

---

## Nós com Query Builder Visual

Os nós de banco de dados (PostgreSQL, MySQL, etc.) oferecem um **query builder visual** além da opção de SQL direto:

### Presets Disponíveis

| Preset | Descrição |
|--------|-----------|
| **Custom SQL** | Escreva SQL livremente |
| **SELECT** | Construtor visual de SELECT |
| **EXISTS** | Verifica se registro existe |
| **COUNT** | Conta registros |
| **ASSERT** | Verifica se valor corresponde |
| **INSERT** | Construtor visual de INSERT |
| **UPDATE** | Construtor visual de UPDATE |
| **DELETE** | Construtor visual de DELETE |

O query builder permite selecionar tabelas, colunas, condições WHERE, ORDER BY e LIMIT sem escrever SQL.

---

## Dicas e Boas Práticas

### Nomeie seus Nós

Dê nomes descritivos aos seus nós alterando o **label**. Isso facilita a leitura do fluxo e torna as expressões mais claras:

```
{{ steps.login.outputs.json.token }}
```
é mais legível que:
```
{{ steps['HTTP Request 2'].outputs.json.token }}
```

### Use Grupos Lógicos

Organize nós relacionados próximos uns dos outros no canvas. Por exemplo, agrupe nós de login em uma área e nós de verificação em outra.

### Configure Evidências

Para testes web, ative screenshots nos passos críticos. Isso facilita a depuração e gera evidências nos relatórios.

### Use Continuar em Falha com Cuidado

O toggle **Continuar em Falha** é útil para nós de verificação (assert) onde você quer registrar todas as falhas em vez de parar na primeira. Use com moderação — em geral, falhas devem interromper o fluxo.

---

## Próximos Passos

- [Execução e Depuração](execucao-depuracao.md) — Como executar e diagnosticar problemas
- [Referência de Nós](../nos/controle-fluxo/if.md) — Detalhes de cada tipo de nó
