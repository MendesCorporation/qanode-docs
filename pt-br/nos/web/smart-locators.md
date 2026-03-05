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

> As opções de Browser & Device são as mesmas do nó Web Flow — consulte a [documentação do Web Flow](web-flow.md#browser) para detalhes sobre presets de viewport e dispositivos mobile disponíveis.

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
| [wait](#wait) | 🟣 | Aguardar condição |
| [scroll](#scroll) | 🟡 | Rolar página |
| [refresh](#refresh) | 🟠 | Recarregar página |
| [frame](#frame) | Cinza | Alternar frame |
| [rightClick](#rightclick) | 🟡 | Clique com botão direito |
| [upload](#upload) | 🔵 | Upload de arquivos |
| [dialog](#dialog) | 🟣 | Tratar dialogs (alert/confirm) |
| [dragDrop](#dragdrop) | 🔵 | Arrastar e soltar |
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

### frame

Alterna para iframe. Suporta modos `name` e `main` (voltar ao principal).

---

## Evidências (Screenshots por Passo)

Idêntico ao Web Flow — cada passo pode ter configuração individual de captura de screenshot.

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `sessionId` | `string` | ID da sessão do navegador |
| `extracts` | `object` | Dados extraídos (chave → valor) |
| `asserts` | `object` | Resultados das asserções (chave → boolean) |

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

## Dicas

- **Prefira `getByRole`** — é o mais resiliente e reflete como assistive technologies encontram elementos
- **Use `getByLabel`** para campos de formulário — é mais estável que CSS
- **`getByTestId`** é ótimo para elementos sem role/label claro
- Use a ação **type** (em vez de fill) para campos com máscaras ou autocomplete
- O campo **Nth** permite selecionar um elemento específico quando múltiplos correspondem ao localizador
