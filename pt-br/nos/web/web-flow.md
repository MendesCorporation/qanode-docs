# Nó Web Flow

O nó **Web Flow** permite automatizar interações com páginas web usando **seletores CSS, XPath, data-testid e texto**. É ideal quando você precisa de seletores tradicionais e controle detalhado sobre a localização de elementos.

> **Dica:** Se preferir localizadores semânticos baseados em acessibilidade (getByRole, getByLabel, etc.), use o nó [Smart Locators](smart-locators.md).

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `web-flow` |
| **Categoria** | Web |
| **Cor** | 🔵 Azul (#3b82f6) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Configuração Geral

| Campo | Tipo | Padrão | Descrição |
|-------|------|--------|-----------|
| **Modo de Sessão** | `new` / `reuse` | `new` | Nova sessão ou reutilizar existente |
| **Session ID** | `string` | — | ID da sessão para reutilizar (quando `reuse`) |
| **Headless** | `boolean` | `true` | Executar sem interface gráfica visível |
| **Estratégia de Storage** | `inMemory` / `persisted` | `inMemory` | Persistir cookies/localStorage entre execuções |
| **Chave de Storage** | `string` | — | Identificador do storage persistido |
| **Browser** | `chromium` / `firefox` / `webkit` | `chromium` | Navegador a usar na execução |
| **Dispositivo** | `desktop` / `mobile` | `desktop` | Modo de emulação |
| **Viewport** | preset ou `custom` | `1920x1080` | Tamanho da janela (modo Desktop) |
| **Largura / Altura** | `number` | — | Dimensões customizadas (quando `custom`) |
| **Modelo de Dispositivo** | string | `iPhone 14` | Dispositivo a emular (modo Mobile) |
| **Varredura de acessibilidade** | `boolean` | `false` | Executar diagnóstico de acessibilidade |

### Browser

| Valor | Navegador |
|-------|-----------|
| `chromium` | Chrome / Chromium (padrão) |
| `firefox` | Firefox |
| `webkit` | Safari / WebKit |

> Firefox e WebKit são baixados automaticamente na primeira execução, caso não estejam instalados.

### Dispositivo — Desktop

Presets de viewport disponíveis:

| Preset | Resolução |
|--------|-----------|
| Full HD (padrão) | 1920 × 1080 |
| 2K | 2560 × 1440 |
| 4K | 3840 × 2160 |
| Laptop | 1440 × 900 |
| HD | 1366 × 768 |
| WXGA | 1280 × 720 |
| XGA | 1024 × 768 |
| Personalizado | largura e altura livres |

### Dispositivo — Mobile

Usa perfis reais do Playwright (viewport, user agent, escala de pixels, touch). Modelos disponíveis:

**iOS:** iPhone SE, iPhone 12, iPhone 13, iPhone 14, iPhone 14 Pro, iPhone 15, iPhone 15 Pro, iPad Mini, iPad (gen 9), iPad Pro 11

**Android:** Pixel 5, Pixel 7, Galaxy S8, Galaxy S9+, Galaxy Tab S4

> No modo Mobile com Firefox, as opções `isMobile` e `hasTouch` são removidas automaticamente (não suportadas pelo Firefox no Playwright). O viewport e o user agent do dispositivo continuam aplicados normalmente.

### Modo de Sessão

- **Nova Sessão (`new`)**: Abre um novo navegador para cada execução. Ideal para testes isolados.
- **Reutilizar Sessão (`reuse`)**: Usa uma sessão de navegador já aberta por outro nó Web Flow/Smart Locators. Útil para dividir testes longos em múltiplos nós mantendo o mesmo navegador e cookies.

### Estratégia de Storage

- **Em Memória (`inMemory`)**: Cookies e localStorage são descartados ao final da execução
- **Persistido (`persisted`)**: Cookies e localStorage são salvos com uma chave e podem ser reutilizados em execuções futuras. Ideal para manter sessões de login entre execuções.

---

## Passos (Web Steps)

O nó Web Flow executa uma sequência de **passos** configuráveis. Cada passo representa uma ação no navegador.

### Ações Disponíveis

| Ação | Cor | Descrição |
|------|-----|-----------|
| [navigate](#navigate) | 🔵 | Navegar para uma URL |
| [click](#click) | 🟡 | Clicar em um elemento |
| [type](#type) | 🟢 | Digitar texto em um campo |
| [wait](#wait) | 🟣 | Aguardar condição ou tempo |
| [assert](#assert) | 🔴 | Verificar condição na página |
| [extract](#extract) | 🔵 Ciano | Extrair dados de um elemento |
| [hover](#hover) | 🔵 Ciano | Passar o mouse sobre um elemento |
| [scroll](#scroll) | 🟡 | Rolar a página |
| [refresh](#refresh) | 🟠 | Recarregar a página |
| [select](#select) | 🟣 | Selecionar opção de um dropdown |
| [press-key](#press-key) | 🟢 | Pressionar uma tecla |
| [frame](#frame) | Cinza | Alternar entre frames/iframes |

---

## Estratégias de Seletor

O Web Flow usa **estratégias de seletor** para localizar elementos na página. Cada passo pode ter múltiplas estratégias ordenadas por prioridade — se a primeira não encontrar o elemento, a próxima é tentada.

| Estratégia | Descrição | Exemplo |
|------------|-----------|---------|
| `css` | Seletor CSS | `#login-button`, `.submit-btn`, `[data-testid="login"]` |
| `dataTestId` | Atributo data-testid | `login-button` |
| `role` | ARIA role | `button` |
| `text` | Texto visível | `Entrar` |
| `xpath` | Expressão XPath | `//button[@type="submit"]` |

**Prioridade recomendada:**
1. `dataTestId` — Mais estável, não muda com layout
2. `css` com ID — Geralmente único na página
3. `role` — Baseado em acessibilidade
4. `text` — Pode mudar com traduções
5. `xpath` — Último recurso, mais frágil

---

## Detalhes das Ações

### navigate

Navega para uma URL específica.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **URL** | `string` | URL de destino (suporta expressões `{{ }}`) |

**Exemplo:**
```
URL: https://meusite.com/login
URL: {{ variables.BASE_URL }}/dashboard
```

---

### click

Clica em um elemento da página.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Seletores** | `array` | Estratégias de seletor |
| **Tentativas** | `number` | Número de tentativas (padrão: 3) |

O click tenta localizar o elemento usando cada estratégia na ordem. Se falhar, aguarda e tenta novamente (até o número de tentativas).

---

### type

Digita texto em um campo de entrada.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Seletores** | `array` | Estratégias de seletor |
| **Texto** | `string` | Texto a ser digitado (suporta `{{ }}`) |
| **Limpar Primeiro** | `boolean` | Limpa o campo antes de digitar |

> **Nota:** A ação `type` usa `fill()` do Playwright, que substitui o valor do campo instantaneamente. Para simular digitação caractere por caractere, use o nó [Smart Locators](smart-locators.md) com a ação `type` (pressSequentially).

---

### wait

Aguarda uma condição antes de prosseguir.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Modo** | `string` | Tipo de espera |
| **Timeout (ms)** | `number` | Tempo máximo de espera |
| **Seletores** | `array` | Seletores (modo `selectorVisible`) |

**Modos:**

| Modo | Descrição |
|------|-----------|
| `networkIdle` | Aguarda todas as requisições de rede terminarem |
| `selectorVisible` | Aguarda um elemento ficar visível |
| `timeout` | Aguarda um tempo fixo (ms) |

---

### assert

Verifica uma condição na página. Se a condição falhar, o passo é marcado como falha.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Nome** | `string` | Identificador da asserção |
| **Modo** | `string` | Tipo de verificação |
| **Seletores** | `array` | Seletores do elemento |
| **Texto Esperado** | `string` | Valor esperado |
| **Case Sensitive** | `boolean` | Sensível a maiúsculas |
| **Continuar em Falha** | `boolean` | Não falhar o nó inteiro |

**Modos:**

| Modo | Descrição |
|------|-----------|
| `elementExists` | Verifica se o elemento existe na página |
| `textContains` | Verifica se o texto do elemento contém o valor esperado |
| `textEquals` | Verifica se o texto do elemento é exatamente o valor esperado |

Os resultados das asserções ficam disponíveis nos outputs:
```
{{ steps["web-flow"].outputs.asserts.nomeAsserção }}  →  true ou false
```

---

### extract

Extrai dados de um elemento da página.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Nome** | `string` | Nome da extração (chave no output) |
| **Seletores** | `array` | Seletores do elemento |
| **Atributo** | `string` | O que extrair |

**Atributos:**

| Atributo | Descrição |
|----------|-----------|
| `text` | Texto visível do elemento |
| `innerHTML` | HTML interno |
| `href` | URL de links |
| `src` | URL de imagens |
| `value` | Valor de campos de formulário |

Os dados extraídos ficam disponíveis nos outputs:
```
{{ steps["web-flow"].outputs.extracts.nomeExtracao }}
```

---

### hover

Passa o mouse sobre um elemento.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Seletores** | `array` | Seletores do elemento |
| **Tentativas** | `number` | Número de tentativas |
| **Espera Após (ms)** | `number` | Tempo de espera após o hover |

---

### scroll

Rola a página ou até um elemento.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Modo** | `string` | Tipo de rolagem |
| **Pixels** | `number` | Quantidade de pixels (modo `by`) |
| **Espera Após (ms)** | `number` | Tempo de espera após rolar |
| **Seletores** | `array` | Seletores (modo `toSelector`) |

**Modos:**

| Modo | Descrição |
|------|-----------|
| `toSelector` | Rola até o elemento ficar visível |
| `by` | Rola uma quantidade fixa de pixels |
| `toBottom` | Rola até o final da página |

---

### refresh

Recarrega a página atual.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Aguardar Até** | `string` | Evento de carregamento |
| **Espera Após (ms)** | `number` | Tempo adicional após recarregar |

**Eventos:**

| Evento | Descrição |
|--------|-----------|
| `load` | Página completamente carregada |
| `domcontentloaded` | DOM pronto (mais rápido) |
| `networkidle` | Sem requisições de rede pendentes |

---

### select

Seleciona uma opção de um elemento `<select>` (dropdown).

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Seletores** | `array` | Seletores do dropdown |
| **Selecionar Por** | `string` | Critério de seleção |
| **Valor** | `string` | Valor correspondente ao critério |

**Critérios:**

| Critério | Descrição |
|----------|-----------|
| `value` | Pelo atributo `value` da option |
| `label` | Pelo texto visível da option |
| `index` | Pelo índice (posição) da option |

---

### press-key

Pressiona uma tecla no teclado.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tecla** | `string` | Nome da tecla (ex: `Enter`, `Tab`, `Escape`) |
| **Seletores** | `array` | Seletores do elemento focado (opcional) |
| **Espera Após (ms)** | `number` | Tempo de espera após pressionar |

---

### frame

Alterna o contexto para um iframe.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Modo** | `string` | Método de seleção do frame |
| **Seletores** | `array` | Seletores (modo `selector`) |
| **Nome do Frame** | `string` | Nome (modo `name`) |
| **Timeout (ms)** | `number` | Tempo máximo de espera |

**Modos:**

| Modo | Descrição |
|------|-----------|
| `selector` | Localiza o iframe por seletor CSS |
| `name` | Localiza o iframe pelo atributo name |
| `main` | Volta ao contexto principal (fora do iframe) |

---

## Evidências (Screenshots)

Cada passo pode ter configuração de evidência individual:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Capturar Screenshot** | `boolean` | Ativar captura |
| **Modo** | `before` / `after` / `both` | Quando capturar |
| **Template de Nome** | `string` | Nome customizado do arquivo |
| **Aguardar Carregamento** | `string` | Evento antes de capturar |
| **Delay (ms)** | `number` | Tempo adicional antes da captura |

---

## Teste de Acessibilidade

O Web Flow suporta escaneamento automático de acessibilidade com **axe-core** integrado. Quando ativado, o nó audita a página após cada passo que captura screenshot, identificando violações WCAG/ARIA e marcando visualmente os elementos problemáticos.

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

### Exemplo: Auditoria de login

```
Configuração do nó:
  Accessibility Scan: true
  Fail When Severity >= : serious

Passos:
1. navigate → https://meusite.com/login    (sem screenshot = sem scan)
2. type → #email → "{{ variables.EMAIL }}" → screenshot after → scan executado
3. click → button[type="submit"]            → screenshot after → scan executado
4. wait → networkIdle
5. assert → loginOk → hasURL → /dashboard  → screenshot after → scan executado
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

## Exemplo Completo

### Login em um site

Passos configurados no nó:

1. **navigate** → `https://meusite.com/login`
2. **type** → Seletor: `#email`, Texto: `{{ variables.USER_EMAIL }}`
3. **type** → Seletor: `#password`, Texto: `{{ variables.USER_PASSWORD }}`
4. **click** → Seletor: `button[type="submit"]`
5. **wait** → Modo: `networkIdle`
6. **assert** → Nome: `loginOk`, Modo: `textContains`, Seletor: `.welcome`, Texto: `Bem-vindo`
7. **extract** → Nome: `userName`, Seletor: `.user-name`, Atributo: `text`

**Resultado:** Após execução, os outputs serão:
```json
{
  "sessionId": "abc-123",
  "extracts": { "userName": "João Silva" },
  "asserts": { "loginOk": true }
}
```

---

## Integração com Custom JavaScript

Após um nó Web Flow ser executado, a sessão do Playwright fica ativa e pode ser acessada diretamente em um nó **Custom JavaScript** posterior no mesmo fluxo.

### Variáveis disponíveis no Custom JS

| Variável | Tipo | Descrição |
|----------|------|-----------|
| `page` | `Page` | Página atual do Playwright (sessão ativa) |
| `context` | `BrowserContext` | Contexto do navegador |
| `browser` | `Browser` | Instância do navegador |
| `web` | `namespace` | Namespace completo com suporte a múltiplas sessões |

> `page`, `context` e `browser` são **atalhos diretos** para a sessão ativa atual. Se não houver sessão ativa, lançam erro ao ser acessados.

### Uso direto via `page`

Acesse a API do Playwright diretamente:

```javascript
// Ler título e URL da página atual
const title = await page.title();
const url = page.url();

// Extrair texto de elemento
const text = await page.locator('.resultado').textContent();

// Executar JavaScript na página
const valor = await page.evaluate(() => document.querySelector('#campo').value);

// Contar elementos
const itens = await page.locator('li.item').count();

return { title, url, text, valor, itens };
```

### Uso via `web.run` (com assertions Playwright)

O método `web.run` injeta o objeto `expect` do Playwright, permitindo assertions nativas:

```javascript
return await web.run(async ({ page, expect, assert }) => {
  // Assertions Playwright
  await expect(page.getByRole('heading')).toContainText('Dashboard');
  await expect(page.locator('.status')).toBeVisible();

  // Ação customizada
  await page.keyboard.press('Escape');

  // Retornar dados
  const count = await page.locator('tr.item').count();
  return { count };
});
```

### Namespace `web` completo

| Método | Descrição |
|--------|-----------|
| `web.current()` | Facade da sessão ativa (inclui `page`, `context`, `browser`, `screenshot()`, etc.) |
| `web.session(id)` | Facade de sessão específica pelo `sessionId` |
| `web.page()` | Retorna o objeto `Page` da sessão ativa |
| `web.context()` | Retorna o `BrowserContext` da sessão ativa |
| `web.browser()` | Retorna o `Browser` da sessão ativa |
| `web.hasSession()` | Retorna `true` se há sessão ativa |
| `web.ids()` | Lista de IDs de sessões ativas |
| `web.sessions()` | Lista de facades de todas as sessões |
| `web.screenshot(nome?, options?)` | Captura screenshot e salva como evidência |
| `web.run(fn, sessionId?)` | Executa função com `{ page, context, browser, expect, assert, ... }` |

### Múltiplas sessões

Quando há mais de um nó Web Flow/Smart Locators aberto no fluxo, cada um tem um `sessionId` distinto. Use `web.session(id)` para acessar uma sessão específica:

```javascript
const sessao1 = web.session(steps["Web Flow 1"].outputs.sessionId);
const sessao2 = web.session(steps["Web Flow 2"].outputs.sessionId);

const url1 = sessao1.url();
const url2 = sessao2.url();

return { url1, url2 };
```

### Exemplo: Validação avançada após login

```
[Web Flow]
  navigate → https://app.com/login
  type → #email → usuario@email.com
  click → #submit
  wait → networkIdle
    │
    ▼
[Custom JavaScript]
  return await web.run(async ({ page, expect }) => {
    // Assert que redirecionou para o dashboard
    await expect(page).toHaveURL(/\/dashboard/);

    // Extrair dados dinâmicos não alcançáveis pelo Web Flow
    const items = await page.evaluate(() =>
      [...document.querySelectorAll('.item')].map(el => el.dataset.id)
    );

    return { items };
  });
```

---

## Dicas

- Use **data-testid** como estratégia principal de seletor — é mais estável
- Configure **screenshots** nos passos críticos para facilitar a depuração
- Use **reutilização de sessão** quando múltiplos nós Web Flow/Smart Locators precisam compartilhar o mesmo navegador
- Para formulários com autocomplete ou máscaras, considere usar o nó [Smart Locators](smart-locators.md) com a ação `type` (digitação caractere por caractere)
- O modo **headless: false** é útil durante o desenvolvimento para visualizar o que o teste está fazendo
