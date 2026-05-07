# Nó Smart Locators

O nó **Smart Locators** permite automatizar interações com páginas web usando **localizadores semânticos do Playwright** — baseados em roles ARIA, labels, placeholders e outros atributos de acessibilidade. Esta abordagem é mais resiliente que seletores CSS, pois reflete como os usuários realmente encontram elementos na página.

> **Dica:** Se preferir seletores CSS e XPath tradicionais, use o nó [Web Flow](web-flow.md).

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `smart-locators` |
| **Categoria** | Web |
| **Cor** | 🔵 Azul (#3b82f6) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Configuração Geral

Idêntica ao nó Web Flow:

| Campo | Tipo | Padrão | Descrição |
|-------|------|--------|-----------|
| **Modo de Sessão** | `new` / `reuse` | `new` | Nova sessão ou reutilizar existente |
| **Session ID** | `string` | — | ID da sessão para reutilizar |
| **Headless** | `boolean` | `true` | Executar sem interface gráfica visível |
| **Estratégia de Storage** | `inMemory` / `persisted` | `inMemory` | Persistir cookies/localStorage |
| **Chave de Storage** | `string` | — | Identificador do storage persistido |
| **Browser** | `chromium` / `firefox` / `webkit` | `chromium` | Navegador a usar na execução |
| **Dispositivo** | `desktop` / `mobile` | `desktop` | Modo de emulação |
| **Viewport** | preset ou `custom` | `1920x1080` | Tamanho da janela (modo Desktop) |
| **Largura / Altura** | `number` | — | Dimensões customizadas (quando `custom`) |
| **Modelo de Dispositivo** | string | `iPhone 14` | Dispositivo a emular (modo Mobile) |
| **Varredura de acessibilidade** | `boolean` | `false` | Executar diagnóstico de acessibilidade |
| **Auditoria de performance** | `boolean` | `false` | Coletar métricas de carregamento e rede durante a execução |
| **Tempo máximo de carregamento (ms)** | `number` | `4000` | Threshold para falhar quando o carregamento da página excede o limite |
| **Tempo máximo de resposta da API (ms)** | `number` | `1500` | Threshold para APIs lentas capturadas na sessão |
| **Falhar em erros de requisição** | `boolean` | `true` | Marca o nó como falho quando houver erro HTTP/rede nas APIs monitoradas |
| **Self Healing** | `boolean` | `false` | Tenta recuperar localizadores quando o elemento muda e o locator original não resolve mais o alvo |

> As opções de Browser & Device são as mesmas do nó Web Flow — consulte a [documentação do Web Flow](web-flow.md#browser) para detalhes sobre presets de viewport e dispositivos mobile disponíveis.

### Auditoria de Performance

Quando **Auditoria de performance** está habilitada, o Smart Locators monitora a sessão Playwright e gera evidências visuais de performance por tela.

O recurso coleta:

- **Page Load** da tela
- **FCP** (*First Contentful Paint*)
- **LCP** (*Largest Contentful Paint*)
- quantidade total de requests
- quantidade de requests de API
- erros de request/API
- APIs lentas por tela

Além do resumo global, o QANode gera **checkpoints por tela navegada**, com:

- nome da tela
- URL
- passo que originou a navegação
- gráfico das APIs mais lentas
- gráfico separado para **erros de API** quando existirem

### Outputs de Performance

Quando a auditoria está ligada, o nó expõe:

- `performancePassed`
- `performanceRequestCount`
- `performanceApiRequestCount`
- `performanceErrorCount`
- `performanceSlowRequestCount`
- `performance`

O objeto `performance` contém um resumo estruturado da execução, incluindo os checkpoints por tela.

### Self Healing

Quando **Self Healing** está habilitado, o Smart Locators tenta recuperar automaticamente ações baseadas em localizadores semânticos quando o locator original deixa de encontrar o elemento esperado.

Além do próprio locator semântico, o mecanismo considera:

- texto e nome acessível do elemento
- `role`
- `label`, `placeholder`, `title` e outros atributos relevantes
- contexto estrutural da página, como container, formulário, heading e posição relativa

Isso ajuda em cenários como:

- mudança de texto com o mesmo significado
- troca entre maiúsculas/minúsculas
- texto com ou sem acento
- variações curtas, como `Entrar` → `Login` ou `Avançar` → `Continuar`

O recurso é conservador: ele só aplica a recuperação quando encontra um candidato claramente mais forte que os demais, evitando “chutes” em telas ambíguas.

Quando um passo é recuperado, a execução mostra:

- o passo afetado
- a confiança da recuperação
- o locator utilizado
- a opção **Aplicar ao Fluxo** para atualizar o fluxo com o locator recuperado

---

## Sistema de Localizadores

### Métodos de Localização

Em vez de seletores CSS, o Smart Locators usa **métodos semânticos** do Playwright:

| Método | Descrição | Exemplo |
|--------|-----------|---------|
| `getByRole` | Localiza por ARIA role | Botão "Entrar", Link "Home" |
| `getByText` | Localiza por texto visível | Texto "Bem-vindo" |
| `getByLabel` | Localiza por label associada | Campo com label "E-mail" |
| `getByPlaceholder` | Localiza por placeholder | Campo com placeholder "Digite seu nome" |
| `getByAltText` | Localiza por alt text | Imagem com alt "Logo" |
| `getByTitle` | Localiza por title | Elemento com title "Fechar" |
| `getByTestId` | Localiza por data-testid | `data-testid="submit-btn"` |

### Configuração do Localizador

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Método** | `string` | Método de localização |
| **Valor** | `string` | Texto/valor a ser buscado |
| **Nome** | `string` | Nome do role (somente para `getByRole`) |
| **Exato** | `boolean` | Correspondência exata (false = contém) |
| **Nth** | `number` | Índice do elemento (quando há múltiplos) |

### ARIA Roles Disponíveis

Para o método `getByRole`, os seguintes roles estão disponíveis:

| Roles de Interação | Roles de Estrutura | Roles de Feedback |
|--------------------|--------------------|-------------------|
| `button` | `heading` | `alert` |
| `checkbox` | `list` | `alertdialog` |
| `combobox` | `listitem` | `dialog` |
| `link` | `navigation` | `progressbar` |
| `listbox` | `region` | `status` |
| `menu` | `row` | `tooltip` |
| `menuitem` | `cell` | — |
| `option` | `grid` | — |
| `radio` | `tab` | — |
| `searchbox` | `tablist` | — |
| `slider` | `tabpanel` | — |
| `spinbutton` | `toolbar` | — |
| `switch` | `tree` | — |
| `textbox` | `treeitem` | — |

**Exemplo de uso:**
```
Método: getByRole
Valor: button
Nome: Entrar
```

Equivale ao Playwright: `page.getByRole('button', { name: 'Entrar' })`

---

## Ações Disponíveis

| Ação | Cor | Descrição |
|------|-----|-----------|
| [navigate](#navigate) | 🔵 | Navegar para URL |
| [click](#click) | 🟡 | Clicar em elemento |
| [dblclick](#dblclick) | 🟡 | Duplo clique |
| [fill](#fill) | 🟢 | Preencher campo (instantâneo) |
| [type](#type) | 🟢 | Digitar caractere por caractere |
| [check](#check) | 🟢 | Marcar checkbox/radio |
| [uncheck](#uncheck) | 🟢 | Desmarcar checkbox |
| [selectOption](#selectoption) | 🟣 | Selecionar opção de dropdown |
| [hover](#hover) | 🔵 Ciano | Passar mouse sobre elemento |
| [focus](#focus) | 🔵 Claro | Focar em elemento |
| [press](#press) | 🟢 | Pressionar tecla |
| [assert](#assert) | 🔴 | Verificar condição |
| [extract](#extract) | 🔵 Ciano | Extrair dados |
| [extractList](#extractlist) | 🟢 Esmeralda | Extrair lista de elementos repetidos |
| [wait](#wait) | 🟣 | Aguardar condição |
| [scroll](#scroll) | 🟡 | Rolar página |
| [refresh](#refresh) | 🟠 | Recarregar página |
| [clock](#clock) | 🟢 Água | Controlar o relógio do navegador |
| [frame](#frame) | Cinza | Alternar frame |
| [rightClick](#rightclick) | 🟡 | Clique com botão direito |
| [upload](#upload) | 🔵 | Upload de arquivos |
| [dialog](#dialog) | 🟣 | Tratar dialogs (alert/confirm) |
| [drag](#drag) | Rosa | Arrastar e soltar |
| [evaluate](#evaluate) | ⚪ | Executar JavaScript na página |
| [screenshot](#screenshot) | 🔵 | Capturar screenshot |

---

## Detalhes das Ações

### navigate

Navega para uma URL.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **URL** | `string` | URL de destino (suporta `{{ }}`) |

---

### click

Clica em um elemento localizado semanticamente.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Localizador** | `locator` | Configuração do localizador |
| **Tentativas** | `number` | Número de tentativas (padrão: 3) |

O click aguarda o elemento ficar visível, rola até ele se necessário, e então clica. Se falhar, tenta novamente.

**Exemplo:**
```
Método: getByRole
Valor: button
Nome: Enviar
Tentativas: 3
```

---

### dblclick

Duplo clique em um elemento.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Localizador** | `locator` | Configuração do localizador |
| **Tentativas** | `number` | Número de tentativas |

---

### fill

Preenche um campo de entrada **instantaneamente** (substitui o valor).

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Localizador** | `locator` | Configuração do localizador |
| **Texto** | `string` | Texto a ser preenchido (suporta `{{ }}`) |
| **Limpar Primeiro** | `boolean` | Limpa o campo antes |

Usa `locator.fill()` do Playwright — ideal para campos simples.

---

### type

Digita texto **caractere por caractere**, simulando digitação real.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Localizador** | `locator` | Configuração do localizador |
| **Texto** | `string` | Texto a ser digitado |
| **Delay (ms)** | `number` | Intervalo entre teclas (padrão: 50ms) |
| **Limpar Primeiro** | `boolean` | Limpa o campo antes |

Usa `locator.pressSequentially()` do Playwright — ideal para campos com:
- **Autocomplete** (sugestões aparecem enquanto digita)
- **Máscaras** (CPF, telefone, CEP)
- **Validação on-keypress** (verifica cada tecla)

**Exemplo:**
```
Localizador: getByPlaceholder → "CPF"
Texto: 123.456.789-00
Delay: 100ms
```

---

### check / uncheck

Marca ou desmarca um checkbox/radio button.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Localizador** | `locator` | Configuração do localizador |

**Exemplo:**
```
Método: getByLabel
Valor: Aceito os termos de uso
```

---

### selectOption

Seleciona uma opção de um dropdown (`<select>`).

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Localizador** | `locator` | Configuração do dropdown |
| **Selecionar Por** | `string` | `value`, `label` ou `index` |
| **Valor** | `string` | Valor correspondente |

---

### hover

Passa o mouse sobre um elemento.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Localizador** | `locator` | Configuração do localizador |
| **Tentativas** | `number` | Número de tentativas |
| **Espera Após (ms)** | `number` | Tempo de espera após hover |

---

### focus

Define o foco em um elemento.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Localizador** | `locator` | Configuração do localizador |

---

### press

Pressiona uma tecla do teclado no contexto de um elemento.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Localizador** | `locator` | Elemento que receberá a tecla |
| **Tecla** | `string` | Nome da tecla (`Enter`, `Tab`, `Escape`, etc.) |
| **Espera Após (ms)** | `number` | Tempo de espera após pressionar |

---

### drag

Arrasta um elemento localizado semanticamente e solta sobre outro localizador ou em coordenadas gravadas.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Localizador de Origem** | `locator` | Elemento que será arrastado |
| **Localizador de Destino** | `locator` | Elemento onde será solto, quando houver |
| **Destino X / Y** | `number` | Coordenadas absolutas exibidas quando o destino não possui localizador confiável |
| **Esperar Após (ms)** | `number` | Tempo de espera após soltar |

O Smart Locators tenta usar os localizadores semânticos do Playwright, como `getByRole`, `getByText`, `getByLabel` e `getByTestId`. Quando a página possui elementos repetidos e o passo vem da extensão, o JSON pode carregar metadados internos de posição para ajudar o executor a escolher o elemento mais próximo do gesto original. Esses metadados não aparecem como campos principais no painel.

Para componentes de drag/drop, o QANode também tenta promover o localizador semântico para o container interativo real. Isso é útil quando o texto clicado fica dentro de outro elemento arrastável, por exemplo:

- Um `<p>` dentro de um card arrastável
- Uma imagem dentro de um container `draggable`
- Uma área de drop marcada por classe ou atributos de drop

Se o destino semântico for genérico demais ou falhar, mas houver `Destino X / Y`, o Smart Locators pode usar as coordenadas gravadas como fallback. Isso permite automatizar canvas, Kanban e áreas de soltar que não têm texto, role ou label próprios.

**Boas práticas:**

- Prefira componentes com nomes acessíveis (`aria-label`, texto visível, labels)
- Use `data-testid` ou `data-qa` para cards, colunas e áreas de drop críticas
- Evite botões apenas com ícone sem `aria-label`
- Revise gravações que dependam de `nth`, principalmente quando há textos repetidos
- Use screenshots `before` e `after` para comprovar visualmente o movimento

---

### assert

Verifica condições na página. Oferece múltiplos modos de verificação:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Nome** | `string` | Identificador da asserção |
| **Localizador** | `locator` | Elemento alvo (quando aplicável) |
| **Modo** | `string` | Tipo de verificação |
| **Texto Esperado** | `string` | Valor esperado |
| **Continuar em Falha** | `boolean` | Não falhar o nó inteiro |

**Modos de Assert:**

| Modo | Descrição | Requer Localizador |
|------|-----------|-------------------|
| `visible` | Elemento está visível | Sim |
| `hidden` | Elemento está oculto | Sim |
| `textContains` | Texto contém valor | Sim |
| `textEquals` | Texto é exatamente o valor | Sim |
| `hasCount` | Contagem de elementos | Sim |
| `enabled` | Elemento está habilitado | Sim |
| `disabled` | Elemento está desabilitado | Sim |
| `checked` | Checkbox/radio está marcado | Sim |
| `hasValue` | Input tem valor específico | Sim |
| `hasAttribute` | Elemento tem atributo com valor | Sim |
| `hasURL` | URL da página contém/é igual a | Não |
| `hasTitle` | Título da página contém/é igual a | Não |

**Exemplos:**

```
// Verificar que botão "Enviar" está visível
Modo: visible, Localizador: getByRole("button", "Enviar")

// Verificar que mensagem contém "Sucesso"
Modo: textContains, Localizador: getByText("Sucesso"), Esperado: "Sucesso"

// Verificar URL após login
Modo: hasURL, Esperado: "/dashboard"

// Verificar que campo está desabilitado
Modo: disabled, Localizador: getByLabel("E-mail")
```

---

### extract

Extrai dados de um elemento.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Nome** | `string` | Nome da extração |
| **Localizador** | `locator` | Configuração do localizador |
| **Atributo** | `string` | O que extrair |

**Atributos disponíveis:**

| Atributo | Descrição |
|----------|-----------|
| `text` | Texto visível (`textContent`) |
| `innerText` | Texto interno |
| `innerHTML` | HTML interno |
| `inputValue` | Valor de campos de input |
| `href` | URL de links |
| `src` | URL de imagens |
| `value` | Valor genérico |

---

### extractList

Itera sobre elementos repetidos na página — linhas de tabela, cards, itens de lista — e extrai um ou mais campos de cada item, produzindo um array de objetos no output.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Nome da Saída** | `string` | Nome da chave no output (ex: `lista_pedidos`) |
| **Seletor dos Itens** | `array` | Estratégias CSS/XPath que identificam cada elemento repetido |
| **Campos** | `array` | Lista de campos a extrair de cada item (ver abaixo) |
| **Limite** | `number` | Número máximo de itens a processar (padrão: 100) |

Cada entrada em **Campos** define:

| Propriedade | Tipo | Descrição |
|-------------|------|-----------|
| `name` | `string` | Nome do campo no objeto de saída |
| `selectorStrategies` | `array` | Seletor relativo dentro do item (`:scope` = o próprio item) |
| `attr` | `string` | Atributo a extrair: `text`, `inputValue`, `innerHTML`, `href`, `src`, etc. |

Os dados extraídos ficam disponíveis nos outputs:
```
{{ steps["smart-locators"].outputs.extracts.lista_pedidos }}
// → [ { numero: "#1042", status: "Enviado" }, { numero: "#1041", status: "Entregue" } ]
```

> Para gravar um passo `extractList` automaticamente pela extensão Chrome, use **Ctrl+Shift+E** durante a gravação.

---

### wait

Aguarda uma condição.

| Modo | Descrição |
|------|-----------|
| `timeout` | Aguarda tempo fixo |
| `visible` | Aguarda elemento ficar visível |
| `hidden` | Aguarda elemento desaparecer |
| `networkIdle` | Aguarda requisições de rede terminarem |

---

### scroll

Rola a página.

| Modo | Descrição |
|------|-----------|
| `toLocator` | Rola até o elemento |
| `by` | Rola quantidade fixa de pixels |
| `toBottom` | Rola até o fim da página |

---

### refresh

Recarrega a página. Mesma configuração do Web Flow.

---

### clock

Controla o relógio do navegador usando `page.clock` do Playwright. A ação é a mesma disponível no Web Flow e pode ser usada para validar comportamentos por data/hora sem depender do horário real da máquina.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Operação** | `string` | `Fixar data/hora`, `Avançar tempo`, `Pausar em` ou `Retomar relógio` |
| **Data / Hora** | `string` | Data/hora alvo para `Fixar data/hora` e `Pausar em` |
| **Duração** | `string` / `number` | Tempo a avançar em `Avançar tempo`, como `20:00`, `02:34:10` ou milissegundos |
| **Recarregar página após aplicar** | `boolean` | Recarrega a página após aplicar o clock quando ativado |
| **Aguardar recarregamento até** | `string` | Evento usado para aguardar o reload |

Novos passos de clock nascem com o reload desligado. Use reload ligado apenas quando a aplicação depende do carregamento da página para recalcular datas ou horários.

---

### frame

Alterna para iframe. Suporta modos `name` e `main` (voltar ao principal).

---

## Evidências (Screenshots por Passo)

Idêntico ao Web Flow — cada passo pode ter configuração individual de captura de screenshot.

---

## Teste de Acessibilidade

O Smart Locators suporta escaneamento automático de acessibilidade com **axe-core** integrado. Quando ativado, o nó audita a página após cada passo que captura screenshot, identificando violações WCAG/ARIA e marcando visualmente os elementos problemáticos.

> O Smart Locators já usa localizadores semânticos baseados em roles ARIA — combiná-lo com o escaneamento de acessibilidade é especialmente útil para verificar se os elementos que você interage passam nos critérios de acessibilidade.

### Configuração

| Campo | Tipo | Padrão | Descrição |
|-------|------|--------|-----------|
| **Accessibility Scan** | `boolean` | `false` | Ativa o escaneamento axe-core |
| **Fail When Severity >=** | `none` / `minor` / `moderate` / `serious` / `critical` | `serious` | Nível mínimo que reprova o nó |

### Como Funciona

A cada passo que tira screenshot, o axe-core é injetado na página e analisa o documento em busca de violações. As violações são desenhadas sobre o screenshot com caixas coloridas no elemento afetado e um painel de contagem no canto:

| Severidade | Cor | Descrição |
|------------|-----|-----------|
| **Critical** | 🔴 Vermelho | Bloqueia acesso a usuários com deficiência |
| **Serious** | 🟠 Laranja | Causa dificuldade significativa |
| **Moderate** | 🟡 Amarelo | Causa alguma dificuldade |
| **Minor** | 🔵 Azul | Melhoria recomendada |

Ao final da execução são gerados automaticamente:

- **Screenshots por passo** — overlay com caixas coloridas + painel `Critical N / Serious N / Moderate N / Minor N` + top 2 regras
- **Gráfico de severidade** — `{id}-accessibility-severity-chart.png` — distribuição por nível
- **Gráfico de regras** — `{id}-accessibility-rule-chart.png` — top 8 regras mais violadas
- **Relatório JSON** — `{id}-accessibility-report.json` — dados completos de todas as violações

### Critério de Aprovação

O nó falha se houver qualquer violação com severidade **igual ou superior** ao threshold configurado. Use `none` para nunca reprovar (coletar métricas sem bloquear o fluxo).

### Exemplo: Verificar acessibilidade do formulário de login

```
Configuração do nó:
  Accessibility Scan: true
  Fail When Severity >= : serious

Passos:
1. navigate → https://meusite.com/login         (sem screenshot = sem scan)
2. fill → getByLabel("E-mail") → "..."           → screenshot after → scan executado
3. fill → getByLabel("Senha") → "..."            → screenshot after → scan executado
4. click → getByRole("button", { name: "Entrar" }) → screenshot after → scan executado
5. wait → networkIdle
6. assert → "loginOk" → textContains → getByText("Bem-vindo") → screenshot → scan executado
```

Se qualquer passo tiver violação `serious` ou `critical`, o nó é reprovado com:
> *"Accessibility findings at or above "serious" were detected."*

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `sessionId` | `string` | ID da sessão do navegador |
| `extracts` | `object` | Dados extraídos (chave → valor) |
| `asserts` | `object` | Resultados das asserções (chave → boolean) |
| `accessibilityPassed` | `boolean` | Se passou no critério de severidade (quando habilitado) |
| `accessibilityViolationCount` | `number` | Total de instâncias de violação encontradas |
| `accessibility` | `object` | Relatório completo de acessibilidade |
| `accessibility.threshold` | `string` | Threshold configurado |
| `accessibility.scanCount` | `number` | Número de checkpoints escaneados |
| `accessibility.counts` | `object` | `{ minor, moderate, serious, critical }` — totais agregados |
| `accessibility.steps` | `array` | Detalhes por checkpoint (url, counts, topRules) |
| `accessibility.rules` | `array` | Top 10 regras violadas em todo o fluxo |

---

## Exemplo Completo: Login com Smart Locators

```
Passo 1: navigate → https://meusite.com/login
Passo 2: fill → getByLabel("E-mail") → "{{ variables.EMAIL }}"
Passo 3: fill → getByLabel("Senha") → "{{ variables.SENHA }}"
Passo 4: click → getByRole("button", { name: "Entrar" })
Passo 5: wait → networkIdle
Passo 6: assert → "loginOk" → textContains → getByText("Bem-vindo")
Passo 7: extract → "userName" → getByRole("heading") → text
```

---

## Quando Usar Smart Locators vs Web Flow

| Cenário | Recomendação |
|---------|-------------|
| Site com boas práticas de acessibilidade | **Smart Locators** |
| Site com data-testid em todos os elementos | **Web Flow** |
| Formulários com labels | **Smart Locators** |
| Elementos sem texto nem role | **Web Flow** (CSS/XPath) |
| Teste de acessibilidade | **Smart Locators** |
| Legacy / sites sem semântica | **Web Flow** |

---

## Integração com Custom JavaScript

O nó **Custom JavaScript** tem acesso direto à sessão do Smart Locators, permitindo estender qualquer fluxo com lógica personalizada, asserções avançadas e manipulação da página usando a API completa do Playwright.

### Variáveis disponíveis no Custom JS

| Variável | Tipo | Descrição |
|----------|------|-----------|
| `page` | `Page` | Atalho direto para a página ativa da sessão atual |
| `context` | `BrowserContext` | Contexto do navegador (cookies, localStorage, múltiplas abas) |
| `browser` | `Browser` | Instância do navegador |
| `web` | `object` | Namespace com métodos para gerenciar sessões web |

> Essas variáveis são populadas automaticamente a partir da sessão ativa. Use um nó Smart Locators antes do Custom JS com `sessionMode: reuse` para compartilhar a sessão.

---

### Uso direto de `page`

Acesse a `page` diretamente para usar qualquer API do Playwright:

```javascript
// Verificar texto com getByRole (mesmo modelo semântico do Smart Locators)
const heading = await page.getByRole('heading', { name: 'Bem-vindo' });
const texto = await heading.textContent();

// Verificar visibilidade
const botao = page.getByRole('button', { name: 'Enviar' });
const visivel = await botao.isVisible();

// Extrair valor de campo localizado por label
const campo = page.getByLabel('E-mail');
const valor = await campo.inputValue();

// Tirar screenshot personalizado
await page.screenshot({ path: '/tmp/resultado.png', fullPage: true });

// Controlar data/hora dentro da mesma sessão Playwright
await page.clock.install({ time: new Date('2026-04-24T10:00:00') });
await page.clock.fastForward('20:00');
```

---

### Asserções com `web.run()`

Para usar o `expect` do Playwright (asserções nativas com retry automático):

```javascript
await web.run(async ({ page, expect }) => {
  // Asserções semânticas — mesmo padrão do Smart Locators
  await expect(page.getByRole('button', { name: 'Enviar' })).toBeVisible();
  await expect(page.getByLabel('E-mail')).toBeEnabled();
  await expect(page.getByText('Sucesso')).toBeVisible();

  // Verificar URL após navegação
  await expect(page).toHaveURL('/dashboard');

  // Verificar título da página
  await expect(page).toHaveTitle(/Painel/);
});
```

O `expect` do Playwright faz retry automático por até 5 segundos, ideal para páginas com animações ou carregamento assíncrono.

---

### Namespace `web`

| Método | Descrição |
|--------|-----------|
| `web.run(async ({ page, expect }) => {})` | Executa bloco com `page` e `expect` da sessão ativa |
| `web.session(id?)` | Retorna `{ page, context, browser }` de uma sessão específica |
| `web.current()` | Retorna `{ page, context, browser }` da sessão mais recente |
| `web.hasSession(id?)` | Verifica se existe sessão ativa (ou com ID específico) |

---

### Múltiplas sessões

Quando o fluxo possui vários nós Smart Locators rodando com sessões diferentes:

```javascript
// Sessão do primeiro nó Smart Locators
const { page: page1 } = web.session('sessao-login');

// Sessão do segundo nó Smart Locators
const { page: page2 } = web.session('sessao-admin');

// Ações em paralelo em sessões diferentes
const [textoUsuario, textoAdmin] = await Promise.all([
  page1.getByRole('heading').textContent(),
  page2.getByRole('heading').textContent(),
]);
```

---

### Exemplo prático: validar resultado de busca semântica

```javascript
// Após nó Smart Locators que realizou uma busca
await web.run(async ({ page, expect }) => {
  // Verificar que lista de resultados está presente
  await expect(page.getByRole('list', { name: 'Resultados' })).toBeVisible();

  // Contar itens
  const itens = page.getByRole('listitem');
  const quantidade = await itens.count();

  if (quantidade === 0) {
    throw new Error('Nenhum resultado encontrado');
  }

  // Verificar que o primeiro resultado contém o termo buscado
  const termo = variables.TERMO_BUSCA;
  await expect(itens.first()).toContainText(termo);
});
```

---

## Dicas

- **Prefira `getByRole`** — é o mais resiliente e reflete como assistive technologies encontram elementos
- **Use `getByLabel`** para campos de formulário — é mais estável que CSS
- **`getByTestId`** é ótimo para elementos sem role/label claro
- Use a ação **type** (em vez de fill) para campos com máscaras ou autocomplete
- O campo **Nth** permite selecionar um elemento específico quando múltiplos correspondem ao localizador
