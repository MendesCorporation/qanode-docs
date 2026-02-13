0l2oed# Extensão Chrome — QANode Recorder

O **QANode Recorder** é uma extensão para Google Chrome que **grava suas interações** com páginas web e as converte automaticamente em passos de teste compatíveis com os nós **Web Flow** ou **Smart Locators** do QANode.

---

## Instalação

1. Abra o Chrome e acesse `chrome://extensions`
2. Ative o **Modo de desenvolvedor** (toggle no canto superior direito)
3. Clique em **Carregar sem compactação**
4. Selecione a pasta `extension/` do QANode
5. A extensão aparecerá na barra de ferramentas

---

## Modo do Recorder

Antes de iniciar a gravação, escolha o **modo do recorder** no popup da extensão:

| Modo | Seletores | Quando Usar |
|------|-----------|-------------|
| **Web Flow** (padrão) | CSS / XPath | Sites com `data-testid`, `id`, ou seletores CSS estáveis |
| **Smart Locators** | Semânticos (getByRole, getByLabel, etc.) | Sites com boas práticas de acessibilidade |

Cada modo gera um nó do mesmo nome no canvas ao colar o JSON.

> **Nota:** Não é possível trocar de modo durante a gravação. Se houver passos gravados, trocar de modo irá limpá-los.

Para mais detalhes sobre as diferenças entre os dois nós, consulte:
- [Web Flow](../nos/web/web-flow.md)
- [Smart Locators](../nos/web/smart-locators.md)

---

## Modos de Gravação

A extensão opera em três modos de interação durante a gravação:

### REC — Gravação Normal

Grava automaticamente suas interações:

| Interação | Ação Gerada |
|-----------|-------------|
| **Clicar** em elemento | `click` com seletores do elemento |
| **Digitar** em campo | `type` com texto e seletores |
| **Selecionar** opção | `select` com valor selecionado |
| **Rolar** a página | `scroll` com posição |
| **Navegar** para URL | `navigate` com URL |
| **Recarregar** página | `refresh` |

### CTRL+A — Modo Assert

Captura elementos para criar **asserções** (verificações):

1. Pressione **Ctrl+A** para ativar o modo assert
2. Clique no elemento a verificar
3. Um passo `assert` é criado com o texto/valor do elemento

### CTRL+E — Modo Extract

Captura elementos para **extração de dados**:

1. Pressione **Ctrl+E** para ativar o modo extract
2. Clique no elemento a extrair
3. Um passo `extract` é criado capturando o texto/valor

---

## Como Usar

### Gravando um Teste

1. Clique no ícone da extensão na barra de ferramentas
2. Selecione o **modo do recorder** (Web Flow ou Smart Locators)
3. Clique em **Start** para iniciar a gravação
4. O indicador fica vermelho indicando gravação ativa
5. Navegue e interaja com o site normalmente
6. Use **Ctrl+A** para adicionar verificações
7. Use **Ctrl+E** para adicionar extrações
8. Clique no ícone novamente e clique em **Stop** para parar

[Popup da extensão](../../assets/images/web-recorder.mp4) 
*Imagem: Popup da extensão mostrando botões REC/STOP, lista de passos gravados e botão Copiar JSON*

### Usando no QANode

1. No popup da extensão, clique em **Copy JSON**
2. No QANode, abra o editor de fluxos
3. Pressione **Ctrl+V** no canvas
4. Um nó será criado automaticamente conforme o modo selecionado:
   - Modo **Web Flow** → nó Web Flow com `selectorStrategies` (CSS/XPath)
   - Modo **Smart Locators** → nó Smart Locators com `locator` (getByRole, getByLabel, etc.)
5. Revise e ajuste os passos conforme necessário

---

## Estratégias de Seletor

### Modo Web Flow

No modo Web Flow, a extensão gera automaticamente **múltiplos seletores CSS/XPath** para cada elemento, ordenados por prioridade:

| Prioridade | Estratégia | Exemplo |
|------------|------------|---------|
| 1 | `data-testid` | `[data-testid="login-btn"]` |
| 2 | `id` | `#submit-button` |
| 3 | `data-qa` | `[data-qa="email-input"]` |
| 4 | `name` (inputs) | `input[name="email"]` |
| 5 | `aria-label` | `[aria-label="Enviar"]` |
| 6 | `placeholder` | `input[placeholder="E-mail"]` |
| 7 | `href` (links) | `a[href="/login"]` |
| 8 | Texto (botões/links) | `button:has-text("Entrar")` |
| 9 | XPath (fallback) | `//button[@type="submit"]` |

O seletor mais específico e estável é sempre priorizado. Isso garante testes mais resilientes.

### Modo Smart Locators

No modo Smart Locators, a extensão converte os elementos capturados em **localizadores semânticos** do Playwright:

| Prioridade | Método | Exemplo |
|------------|--------|---------|
| 1 | `getByTestId` | Elemento com `data-testid="submit-btn"` |
| 2 | `getByLabel` | Campo com `aria-label="E-mail"` |
| 3 | `getByPlaceholder` | Campo com `placeholder="Digite seu nome"` |
| 4 | `getByText` | Elemento com texto visível |
| 5 | `getByRole` | Botão, link, heading (fallback) |

Os localizadores semânticos são mais resilientes a mudanças de layout e refletem como os usuários realmente encontram elementos na página.

---

## Recursos Inteligentes

### Deduplicação

A extensão evita duplicação de passos:

- **Click + Navegação**: Se um click dispara uma navegação, o passo `navigate` extra é suprimido
- **Type + Type**: Digitação consecutiva no mesmo campo é consolidada em um único passo
- **Scroll**: Rolagens pequenas (< 200px) são ignoradas

### Formulários

- **Focus out**: Salva o valor do input quando o campo perde o foco
- **Enter**: Salva o input e registra o click no botão submit
- **Change**: Salva imediatamente para dropdowns e selects

### Persistência

A gravação sobrevive a **recarregamentos de página**:
- Os passos são salvos no `chrome.storage.local`
- Ao recarregar, a gravação continua do ponto onde parou
- Um passo `refresh` é automaticamente adicionado

### Detecção de Navegação

A extensão usa o Performance API para distinguir entre:
- **Navegação** (nova URL) → gera passo `navigate`
- **Refresh** (mesma URL) → gera passo `refresh`
- **Click → Navegação** (link clicado) → suprime o `navigate` redundante

---

## Formato de Exportação

O JSON copiado é compatível com o nó correspondente ao modo selecionado.

### Web Flow → Nó Web Flow

```json
{
  "type": "web-flow",
  "position": { "x": 100, "y": 100 },
  "data": {
    "label": "Recorded Flow",
    "sessionMode": "new",
    "headless": true,
    "storageStrategy": "inMemory",
    "webSteps": [
      {
        "id": "step-1",
        "action": "navigate",
        "label": "Navigate to page",
        "config": {
          "url": "https://exemplo.com/login"
        },
        "evidence": {
          "takeScreenshot": true,
          "screenshotMode": "after"
        }
      },
      {
        "id": "step-2",
        "action": "type",
        "label": "Type \"user@email.com\"",
        "config": {
          "selectorStrategies": [
            { "type": "css", "value": "#email" },
            { "type": "css", "value": "input[name='email']" }
          ],
          "text": "user@email.com",
          "clearFirst": true
        }
      },
      {
        "id": "step-3",
        "action": "click",
        "label": "Click \"Entrar\"",
        "config": {
          "selectorStrategies": [
            { "type": "css", "value": "[data-testid='login-btn']" },
            { "type": "css", "value": "button[type='submit']" }
          ]
        }
      }
    ]
  }
}
```

### Smart Locators → Nó Smart Locators

```json
{
  "type": "smart-locators",
  "position": { "x": 100, "y": 100 },
  "data": {
    "label": "Recorded Smart Locators",
    "sessionMode": "new",
    "headless": true,
    "storageStrategy": "inMemory",
    "smartSteps": [
      {
        "id": "step-1",
        "action": "navigate",
        "label": "Navigate to page",
        "config": {
          "url": "https://exemplo.com/login"
        },
        "evidence": {
          "takeScreenshot": true,
          "screenshotMode": "after"
        }
      },
      {
        "id": "step-2",
        "action": "fill",
        "label": "Fill \"user@email.com\"",
        "config": {
          "locator": {
            "method": "getByLabel",
            "value": "E-mail",
            "exact": true
          },
          "text": "user@email.com",
          "clearFirst": false
        }
      },
      {
        "id": "step-3",
        "action": "click",
        "label": "Click \"Entrar\"",
        "config": {
          "locator": {
            "method": "getByRole",
            "value": "button",
            "name": "Entrar",
            "exact": false
          }
        }
      }
    ]
  }
}
```

> **Diferenças principais:** No modo Smart Locators, a ação `type` é convertida para `fill`, `select` para `selectOption`, e os seletores CSS são substituídos por localizadores semânticos (`locator` em vez de `selectorStrategies`).

---

## Fluxo de Trabalho Recomendado

```
1. Escolha o modo do recorder (Web Flow ou Smart Locators)
2. Grave a interação com a extensão
3. Cole no QANode (Ctrl+V)
4. Revise os passos gerados
5. Adicione waits onde necessário (a extensão não grava esperas)
6. Adicione asserts para validações (ou grave com Ctrl+A)
7. Ajuste seletores/localizadores se necessário
8. Configure screenshots nos passos críticos
9. Salve e execute
```

---

## Limitações

- **Apenas Chrome** — não funciona em outros navegadores
- **Sem waits automáticos** — adicione manualmente no QANode
- **Sem drag & drop** — não grava operações de arrastar
- **Sites com CSP restrito** — podem bloquear a injeção de content script
- **iFrames cross-origin** — podem não ser acessíveis

---

## Dicas

- **Escolha o modo certo** — use Smart Locators para sites acessíveis, Web Flow para sites com seletores CSS estáveis
- **Grave ações simples** e refine no QANode
- **Use Ctrl+A** (assert) durante a gravação em pontos de verificação
- **Use Ctrl+E** (extract) para capturar dados que serão usados em nós posteriores
- **Adicione waits** no QANode para elementos que demoram a carregar
- **Revise os seletores** — no modo Web Flow, prefira `data-testid`; no modo Smart Locators, prefira `getByRole` ou `getByLabel`
- **Grave em segmentos** — para fluxos longos, grave por partes e combine no QANode
