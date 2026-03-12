# Nó Custom JavaScript

O nó **Custom JavaScript** permite executar código JavaScript customizado durante a execução do fluxo. Ideal para transformações de dados, cálculos complexos e lógica que não é coberta pelos nós nativos.

Além do JavaScript puro, o nó também pode trabalhar com sessões já abertas de **Web** e **Mobile**, permitindo executar automações avançadas com **Playwright** e **Appium** diretamente no código.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `custom-js` |
| **Categoria** | Utilitários |
| **Cor** | ⚪ Cinza (#6b7280) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Configuração

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Código** | `string` | Código JavaScript a ser executado |

O editor de código usa o **Monaco Editor** (mesmo do VS Code) com:
- Syntax highlighting
- Autocomplete para `steps`, `variables`, `web`, `mobile`, `page`, `app`, `return`, `console.log`
- Validação de sintaxe

---

## Contexto de Execução

O código tem acesso a:

| Variável | Tipo | Descrição |
|----------|------|-----------|
| `steps` | `object` | Outputs de todos os nós anteriores |
| `variables` | `object` | Variáveis globais e de runtime |
| `expect` | `function` | Assertions do Playwright para cenários Web |
| `assert` | `object` | Assertions do Node.js |
| `fetch` | `function` | Requisições HTTP diretas via JavaScript |
| `web` | `object` | Namespace para trabalhar com sessões Web já abertas |
| `mobile` | `object` | Namespace para trabalhar com sessões Mobile já abertas |
| `page` | `object` | Alias global da página Web atual |
| `context` | `object` | Alias global do contexto Web atual |
| `browser` | `object` | Alias global do browser atual |
| `app` | `object` | Alias global da sessão Mobile atual |
| `client` | `object` | Alias de compatibilidade para `app` |

### Acessando dados de nós anteriores

```javascript
// Output de um HTTP Request
const userData = steps["http-request"].outputs.json;
const userName = userData.name;

// Output de uma query PostgreSQL
const rows = steps["postgres-query"].outputs.rows;
const firstUser = rows[0];

// Output de extração web
const title = steps["web-flow"].outputs.extracts.pageTitle;
```

### Acessando variáveis

```javascript
const apiUrl = variables.API_URL;
const environment = variables.ENVIRONMENT;
```

---

## Sessões Web e Mobile

O **Custom JavaScript** não cria automaticamente uma sessão Web ou Mobile. Ele usa uma sessão que já foi aberta por um nó anterior.

### Sessão Web

Para usar `web`, `page`, `context` ou `browser`, você precisa ter uma sessão Web ativa criada por um nó como:
- **Web Flow**
- **Smart Locators**

### Sessão Mobile

Para usar `mobile`, `app` ou `client`, você precisa ter uma sessão Mobile ativa criada por um nó como:
- **Mobile Flow**

### Regra prática

- Use `web` quando quiser avançar uma automação Web já aberta
- Use `mobile` quando quiser avançar uma automação Mobile já aberta
- Se não houver sessão ativa, essas APIs falharão

---

## Automação Web com Playwright

O namespace `web` expõe a sessão Web atual e utilitários avançados baseados em **Playwright**.

### Métodos do namespace `web`

| Método | Descrição |
|--------|-----------|
| `web.hasSession(sessionId?)` | Verifica se existe uma sessão Web ativa |
| `web.ids()` | Retorna os IDs das sessões Web disponíveis |
| `web.sessions()` | Retorna todas as sessões Web disponíveis |
| `web.current()` | Retorna a sessão Web atual |
| `web.session(sessionId)` | Retorna uma sessão Web específica |
| `web.page(sessionId?)` | Retorna a `page` do Playwright |
| `web.context(sessionId?)` | Retorna o `context` do Playwright |
| `web.browser(sessionId?)` | Retorna o `browser` do Playwright |
| `web.screenshot(name?, options?, sessionId?)` | Captura screenshot da sessão Web |
| `web.run(fn, sessionId?)` | Executa código avançado na sessão Web |

### Objeto retornado por `web.current()` e `web.session()`

| Campo / Método | Descrição |
|----------------|-----------|
| `id` | ID interno da sessão |
| `sessionId` | ID público da sessão |
| `page` | Instância Playwright `page` |
| `context` | Instância Playwright `context` |
| `browser` | Instância Playwright `browser` |
| `session` | Objeto bruto da sessão |
| `storageStrategy` | Estratégia de storage da sessão |
| `persistedKey` | Chave de persistência da sessão |
| `url()` | Retorna a URL atual |
| `title()` | Retorna o título atual |
| `locator(selector)` | Cria um locator Playwright |
| `screenshot(name?, options?)` | Tira screenshot dessa sessão |

### Métodos mais usados em `page`

O objeto `page` é uma instância Playwright. Alguns métodos úteis:

| Método | Descrição |
|--------|-----------|
| `page.goto(url, options?)` | Navega para uma URL |
| `page.url()` | Retorna a URL atual |
| `page.title()` | Retorna o título atual |
| `page.reload()` | Recarrega a página |
| `page.goBack()` | Volta no histórico |
| `page.goForward()` | Avança no histórico |
| `page.locator(selector)` | Cria um locator |
| `page.getByRole(...)` | Busca por role acessível |
| `page.getByText(...)` | Busca por texto |
| `page.getByLabel(...)` | Busca por label |
| `page.getByPlaceholder(...)` | Busca por placeholder |
| `page.getByTestId(...)` | Busca por `data-testid` |
| `page.waitForURL(...)` | Aguarda mudança de URL |
| `page.waitForResponse(...)` | Aguarda uma resposta de rede |
| `page.waitForSelector(...)` | Aguarda um seletor aparecer |
| `page.evaluate(...)` | Executa JavaScript dentro da página |
| `page.screenshot(...)` | Captura screenshot via Playwright |

### Screenshot Web

Por padrão, `web.screenshot()` captura apenas o **viewport atual**, seguindo o mesmo padrão dos nós Web.

```javascript
await web.screenshot("pagina-atual");
```

Para capturar a página inteira:

```javascript
await web.screenshot("pagina-completa", { fullPage: true });
```

### `web.run(...)`

`web.run(...)` é a forma recomendada para executar código avançado sobre a sessão Web. O callback recebe:

| Campo | Descrição |
|-------|-----------|
| `page` | Página atual |
| `context` | Contexto atual |
| `browser` | Browser atual |
| `session` | Sessão atual |
| `sessionId` | ID da sessão |
| `id` | ID interno da sessão |
| `storageStrategy` | Estratégia de storage |
| `persistedKey` | Chave de persistência |
| `url` | Helper para URL atual |
| `title` | Helper para título atual |
| `locator` | Helper para locators |
| `screenshot` | Helper para screenshots |
| `expect` | Assertions Playwright |
| `assert` | Assertions Node.js |

### Exemplo Web: validar produtos e tirar screenshot

```javascript
return await web.run(async ({ page, expect }) => {
  await page.goto('https://automationexercise.com/products', {
    waitUntil: 'domcontentloaded'
  });

  const products = page.locator('.features_items .product-image-wrapper');
  await expect(products).toHaveCount(34);

  await web.screenshot('web-products');

  return {
    url: page.url(),
    title: await page.title(),
    productCount: await products.count()
  };
});
```

### Exemplo Web: usar a sessão atual sem navegar

```javascript
return await web.run(async ({ page }) => {
  return {
    url: page.url(),
    title: await page.title()
  };
});
```

### Exemplo Web: usar o alias global `page`

```javascript
await page.goto('https://example.com');
await web.screenshot('example-home');

return {
  title: await page.title(),
  url: page.url()
};
```

---

## Automação Mobile com Appium

O namespace `mobile` expõe a sessão Mobile atual e utilitários avançados baseados em **Appium**.

### Métodos do namespace `mobile`

| Método | Descrição |
|--------|-----------|
| `mobile.hasSession(sessionId?)` | Verifica se existe uma sessão Mobile ativa |
| `mobile.ids()` | Retorna os IDs das sessões Mobile disponíveis |
| `mobile.sessions()` | Retorna todas as sessões Mobile disponíveis |
| `mobile.current()` | Retorna a sessão Mobile atual |
| `mobile.session(sessionId)` | Retorna uma sessão Mobile específica |
| `mobile.app(sessionId?)` | Retorna o helper Mobile principal |
| `mobile.client(sessionId?)` | Alias de compatibilidade para `mobile.app()` |
| `mobile.screenshot(name?, sessionId?)` | Captura screenshot da sessão Mobile |
| `mobile.source(sessionId?)` | Retorna o XML/source da tela atual |
| `mobile.run(fn, sessionId?)` | Executa código avançado na sessão Mobile |

### Objeto retornado por `mobile.current()` e `mobile.session()`

| Campo / Método | Descrição |
|----------------|-----------|
| `id` | ID interno da sessão |
| `sessionId` | ID público da sessão |
| `app` | Helper Mobile principal |
| `client` | Alias de compatibilidade para `app` |
| `capabilities` | Capabilities da sessão |
| `session` | Objeto bruto da sessão |
| `screenshot(name?)` | Tira screenshot da sessão |
| `source()` | Retorna o source/XML da tela |
| `request(method, path, body?)` | Faz chamada direta para a sessão Appium |
| `executeScript(script, args?)` | Executa script Appium |

### `app` e `client`

O nome principal para Mobile é **`app`**.  
`client` continua existindo por compatibilidade, mas o uso recomendado é `app`.

### Métodos expostos em `app`

| Método | Descrição |
|--------|-----------|
| `createSession(capabilities)` | Cria uma nova sessão Appium |
| `deleteSession(sessionId)` | Encerra uma sessão |
| `screenshot(sessionId)` | Captura screenshot da sessão |
| `getPageSource(sessionId)` | Retorna o XML/source da tela |
| `findElement(sessionId, using, value)` | Busca um elemento |
| `findElements(sessionId, using, value)` | Busca vários elementos |
| `elementClick(sessionId, elementId)` | Clica em um elemento |
| `elementSendKeys(sessionId, elementId, text)` | Digita em um elemento |
| `sendKeys(sessionId, text)` | Envia texto diretamente para a sessão |
| `elementClear(sessionId, elementId)` | Limpa um campo |
| `elementText(sessionId, elementId)` | Retorna o texto do elemento |
| `elementAttribute(sessionId, elementId, name)` | Retorna atributo do elemento |
| `elementDisplayed(sessionId, elementId)` | Verifica se o elemento está visível |
| `tapAt(sessionId, x, y)` | Toca em coordenadas |
| `doubleTapAt(sessionId, x, y)` | Executa double tap em coordenadas |
| `swipe(sessionId, startX, startY, endX, endY, durationMs?)` | Executa swipe |
| `pressKeyCode(sessionId, keyCode)` | Pressiona keycode Android |
| `req(method, path, body?)` | Faz chamada bruta para o Appium |
| `executeScript(sessionId, script, args?)` | Executa script Appium |
| `launchApp(sessionId)` | Abre o app novamente |
| `resetApp(sessionId)` | Reseta o app |

### Screenshot Mobile

```javascript
await mobile.screenshot('mobile-current-screen');
```

### `mobile.run(...)`

`mobile.run(...)` é a forma recomendada para executar código avançado sobre a sessão Mobile. O callback recebe:

| Campo | Descrição |
|-------|-----------|
| `app` | Helper Mobile principal |
| `client` | Alias para `app` |
| `sessionId` | ID da sessão |
| `id` | ID interno da sessão |
| `capabilities` | Capabilities atuais |
| `session` | Sessão atual |
| `screenshot` | Helper para screenshots |
| `source` | Helper para source |
| `request` | Helper para chamadas diretas |
| `executeScript` | Helper para scripts Appium |
| `assert` | Assertions Node.js |

### Exemplo Mobile: tocar por coordenada e tirar screenshot

```javascript
return await mobile.run(async ({ app, sessionId }) => {
  const source = await app.getPageSource(sessionId);

  await app.tapAt(sessionId, 200, 400);
  await mobile.screenshot('mobile-after-tap');

  return {
    sessionId,
    sourceLength: source.length,
    tapped: true
  };
});
```

### Exemplo Mobile: clicar por `accessibility id`

```javascript
return await mobile.run(async ({ app, sessionId }) => {
  const elementId = await app.findElement(sessionId, 'accessibility id', 'login-button');
  await app.elementClick(sessionId, elementId);

  await mobile.screenshot('login-button-clicked');

  return {
    clicked: true,
    elementId
  };
});
```

### Exemplo Mobile: usar o alias global `app`

```javascript
const current = mobile.current();
const source = await app.getPageSource(current.sessionId);

return {
  sessionId: current.sessionId,
  sourceLength: source.length
};
```

---

## Retornando Valores

O código **deve retornar** um valor usando `return`. O valor retornado fica disponível nos outputs:

```javascript
// Retornar valor simples
return "Hello World";

// Retornar objeto
return {
  fullName: steps["Query"].outputs.rows[0].first_name + " " + steps["Query"].outputs.rows[0].last_name,
  isAdmin: steps["Query"].outputs.rows[0].role === "admin"
};

// Retornar array
return steps["Query"].outputs.rows.map(row => row.email);
```

### Retornando o resultado de `web.run()` e `mobile.run()`

Se você quiser que o valor retornado dentro de `web.run(...)` ou `mobile.run(...)` apareça em `outputs.result`, o script principal também precisa retornar esse resultado:

```javascript
const result = await web.run(async ({ page }) => {
  return { url: page.url() };
});

return result;
```

Ou diretamente:

```javascript
return await mobile.run(async ({ app, sessionId }) => {
  await app.tapAt(sessionId, 200, 400);
  return { tapped: true };
});
```

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `result` | `any` | Valor retornado pelo código |

### Acessando o resultado

```
{{ steps["custom-js"].outputs.result }}
{{ steps["custom-js"].outputs.result.fullName }}
{{ steps["custom-js"].outputs.result[0] }}
```

---

## Exemplos Práticos

### Transformar dados de API

```javascript
const items = steps["http-request"].outputs.json.items;

// Filtrar e transformar
const activeItems = items
  .filter(item => item.active)
  .map(item => ({
    id: item.id,
    name: item.name.toUpperCase(),
    price: (item.price * 1.1).toFixed(2)
  }));

return {
  items: activeItems,
  count: activeItems.length,
  totalPrice: activeItems.reduce((sum, item) => sum + parseFloat(item.price), 0)
};
```

### Gerar dados de teste

```javascript
const timestamp = Date.now();
return {
  email: `test-${timestamp}@exemplo.com`,
  username: `user_${timestamp}`,
  password: `Pass${timestamp}!`
};
```

### Validação complexa

```javascript
const response = steps["API Check"].outputs.json;
const dbRecord = steps["DB Query"].outputs.rows[0];

const validations = [];

if (response.name !== dbRecord.name) {
  validations.push(`Nome diverge: API="${response.name}" DB="${dbRecord.name}"`);
}

if (response.email !== dbRecord.email) {
  validations.push(`Email diverge: API="${response.email}" DB="${dbRecord.email}"`);
}

return {
  valid: validations.length === 0,
  errors: validations,
  errorCount: validations.length
};
```

### Formatar relatório

```javascript
const runs = steps["Query Runs"].outputs.rows;

const summary = {
  total: runs.length,
  passed: runs.filter(r => r.status === 'success').length,
  failed: runs.filter(r => r.status === 'failed').length,
  passRate: 0
};

summary.passRate = ((summary.passed / summary.total) * 100).toFixed(1) + '%';

return summary;
```

### JavaScript puro

```javascript
const now = new Date().toISOString();
const total = [10, 20, 30].reduce((sum, value) => sum + value, 0);

return {
  now,
  total,
  average: total / 3
};
```

---

## Limitações

- O código roda em um **ambiente sandbox**
- Não há acesso a:
  - Sistema de arquivos (`fs`)
  - Módulos Node.js via `require` / `import`
  - `process.env` diretamente
- Para variáveis de ambiente, use `variables`
- Para interações Web e Mobile, é necessário já existir uma sessão ativa
- `client` continua suportado, mas o nome recomendado para Mobile é `app`

> `fetch` está disponível. Ainda assim, para fluxos HTTP estruturados e mais fáceis de manter, prefira o nó **HTTP Request**.

---

## Dicas

- Use nomes descritivos para o nó: "Transformar Dados", "Gerar Credenciais", "Validar Resposta", "Código Web Avançado", "Código Mobile Avançado"
- **Sempre retorne** um valor — sem `return`, o output será `undefined`
- Para depuração, use `console.log()` — as mensagens aparecem nos logs do nó
- O Monaco Editor oferece autocomplete — digite `steps.`, `web.` ou `mobile.` para ver as opções disponíveis
- Para automação Web avançada, prefira `web.run(...)`
- Para automação Mobile avançada, prefira `mobile.run(...)`
- Use `web.screenshot()` e `mobile.screenshot()` quando quiser evidências adicionais
- Use para **transformar dados** entre nós e para resolver cenários avançados que não foram cobertos pelos nós visuais
