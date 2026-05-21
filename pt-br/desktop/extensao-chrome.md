# Extensão Chrome — QANode Recorder

O **QANode Recorder** é uma extensão para Google Chrome que **grava suas interações** com páginas web e as converte automaticamente em passos de teste compatíveis com os nós **Smart Web Flow**, **Web Flow** ou **Smart Locators** do QANode.

---

## Instalação

Instale pela Chrome Web Store:

[QANode Recorder na Chrome Web Store](https://chromewebstore.google.com/detail/qanode-recorder/gmdklmpafacfkjmljgnoafcjjcdjiajo)

Depois de instalar:

1. Fixe o ícone do QANode Recorder na barra do Chrome
2. Abra o site que deseja gravar
3. Clique no ícone da extensão
4. Escolha o modo de gravação
5. Clique em **Start**

Em ambientes de desenvolvimento interno, também é possível instalar manualmente por `chrome://extensions` usando **Carregar sem compactação**, quando orientado pelo time QANode.

---

## Modo do Recorder

Antes de iniciar a gravação, escolha o **modo do recorder** no popup da extensão:

| Modo | Estratégia | Quando Usar |
|------|------------|-------------|
| **Smart Web Flow** (recomendado) | Semântica + CSS/XPath + identidade + contexto | Novos fluxos web, aplicações enterprise, menus, modais, grids, iframes, drag/drop e páginas dinâmicas |
| **Web Flow** | CSS / XPath | Fluxos legados ou sites com `data-testid`, `id` e seletores CSS muito estáveis |
| **Smart Locators** | Semânticos (`getByRole`, `getByLabel`, etc.) | Fluxos legados baseados em acessibilidade e páginas mais simples |
| **XPath Inspector** | Candidatos XPath | Diagnóstico manual de seletores, sem gerar nó de fluxo |

Cada modo gera um nó do mesmo nome no canvas ao colar o JSON.

> **Nota:** Não é possível trocar de modo durante a gravação. Se houver passos gravados, trocar de modo irá limpá-los.

Para mais detalhes sobre as diferenças entre os nós web, consulte:
- [Smart Web Flow](../nos/web/smart-web-flow.md)
- [Web Flow](../nos/web/web-flow.md)
- [Smart Locators](../nos/web/smart-locators.md)

---

## Modos de Gravação

A extensão opera em modos de interação durante a gravação:

### REC — Gravação Normal

Grava automaticamente suas interações:

| Interação | Ação Gerada |
|-----------|-------------|
| **Clicar** em elemento | `click` com seletores do elemento |
| **Digitar** em campo | `fill`/`type` com texto e alvo |
| **Selecionar** opção | `selectOption`/`select` com valor selecionado |
| **Rolar** a página | `scroll` com posição |
| **Arrastar e soltar** elemento | `drag` com origem, destino e/ou coordenadas |
| **Navegar** para URL | `navigate` com URL |
| **Recarregar** página | `refresh` |

No modo **Smart Web Flow**, as ações gravadas também podem carregar metadados adicionais, como identidade do alvo, texto de escopo, estratégias alternativas, contexto de iframe, efeitos esperados e evidências.

Quando um click abre uma nova aba ou janela, a extensão pode inserir um passo **Trocar página / aba** (`switchPage`) depois do click. Assim os próximos passos continuam na aba correta em vez de tentar agir na página anterior.

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

### CTRL+SHIFT+E — Modo Extract List

Captura **listas de elementos repetidos** (linhas de tabela, cards, itens de lista) em dois passos:

1. Pressione **Ctrl+Shift+E** para ativar o modo de extração de lista
2. O indicador muda para **LIST:ITEM** — clique no elemento repetido que representa um item da lista (ex: uma linha de tabela, um card)
3. Overlays ciano destacam todos os itens detectados com o mesmo padrão
4. O indicador muda para **LIST:0** — clique nos campos que deseja extrair dentro de cada item (ex: coluna nome, coluna preço)
5. Cada campo clicado é destacado em laranja e adicionado ao passo. O contador no indicador aumenta a cada campo
6. Clique no **indicador REC** para finalizar — um passo `extractList` é gravado com todos os campos selecionados

> Pressionar **Ctrl+A** ou **Ctrl+E** durante a captura de lista cancela o modo e retorna ao modo normal.

### CTRL+ALT+W — Modo Wait

Cria uma espera explícita para um elemento da página.

1. Pressione **Ctrl+Alt+W** para ativar o modo wait
2. Clique no elemento que precisa estar visível antes de continuar
3. Um passo `wait` é criado com o alvo selecionado

Use este modo quando a página tem carregamentos assíncronos, componentes que aparecem depois de uma animação ou elementos que dependem de API. O Smart Web Flow já grava alguns efeitos esperados automaticamente, mas waits explícitos continuam úteis para pontos críticos do fluxo.

### CTRL+ALT+T — Modo Extract Table

Disponível no modo **Smart Web Flow**, captura uma **tabela ou grid** como dados tabulares estruturados.

1. Pressione **Ctrl+Alt+T** para ativar o modo de extração de tabela
2. Clique na tabela, grid ou região tabular que deseja capturar
3. O QANode Recorder cria um passo `extractTable`
4. No QANode, revise o nome da variável de saída e o limite de linhas, se necessário

Use este modo quando a página já apresenta dados em formato de tabela. Para cards, listas visuais ou tabelas muito customizadas, o modo **Extract List** pode ser melhor, pois permite escolher manualmente quais campos de cada item serão extraídos.

### XPath Inspector

O modo **XPath Inspector** ajuda a investigar seletores diretamente na página. Ele não grava um fluxo de teste; ele captura candidatos XPath para o elemento clicado.

1. Selecione **XPath Inspector** no popup.
2. Clique em **Activate**.
3. Passe pela página para ver os elementos visíveis destacados.
4. Clique no elemento que deseja inspecionar.
5. A extensão mostra os XPaths encontrados e indica quais parecem únicos.
6. Copie o XPath desejado para usar manualmente quando necessário.

Ao clicar em um elemento, o QANode Recorder também pisca o alvo selecionado na página. Isso ajuda a confirmar se o XPath corresponde ao elemento correto.

---

## Como Usar

### Gravando um Teste

1. Clique no ícone da extensão na barra de ferramentas
2. Selecione o **modo do recorder** (Smart Web Flow, Web Flow ou Smart Locators)
3. Clique em **Start** para iniciar a gravação
4. O indicador fica vermelho indicando gravação ativa
5. Navegue e interaja com o site normalmente
6. Use **Ctrl+A** para adicionar verificações
7. Use **Ctrl+E** para adicionar extrações de valor único
8. Use **Ctrl+Shift+E** para adicionar extrações de lista (elementos repetidos)
9. Use **Ctrl+Alt+W** para adicionar waits explícitos
10. Use **Ctrl+Alt+T** para adicionar extração de tabela no modo Smart Web Flow
11. Clique no ícone novamente e clique em **Stop** para parar

[Popup da extensão](../../assets/images/web-recorder.mp4) 
*Imagem: Popup da extensão mostrando botões REC/STOP, lista de passos gravados e botão Copiar JSON*

### Usando no QANode

1. No popup da extensão, clique em **Copy JSON**
2. No QANode, abra o editor de fluxos
3. Pressione **Ctrl+V** no canvas
4. Um nó será criado automaticamente conforme o modo selecionado:
   - Modo **Smart Web Flow** → nó Smart Web Flow com `smartWebSteps`
   - Modo **Web Flow** → nó Web Flow com `selectorStrategies` (CSS/XPath)
   - Modo **Smart Locators** → nó Smart Locators com `locator` (getByRole, getByLabel, etc.)
5. Revise e ajuste os passos conforme necessário

---

## Estratégias de Seletor

### Modo Smart Web Flow

No modo Smart Web Flow, a extensão grava mais do que um seletor. Cada passo pode incluir:

- localizador semântico principal;
- seletores CSS/XPath alternativos;
- identidade do elemento clicado/preenchido;
- texto, role, tag e atributos úteis;
- contexto de linha, card ou item repetido;
- informações de iframe;
- posição aproximada do elemento;
- efeitos esperados após a ação.

Isso permite que o QANode execute o passo com mais contexto e evite depender apenas de um seletor frágil.

Quando a extensão detecta que o alvo está dentro de uma linha, card ou item repetido, ela pode gravar um **Texto do escopo**. Esse texto ajuda o Smart Web Flow a encontrar primeiro o item correto e só depois o botão, link, campo ou controle dentro dele.

Exemplo:

```
Texto do escopo: INC0010008
Alvo: botão "Abrir"
```

Nesse caso, o QANode procura o item `INC0010008` e clica no botão **Abrir** dentro desse item, em vez de clicar no primeiro botão **Abrir** da tela.

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

## Arrastar e Soltar

A extensão também grava operações de **arrastar e soltar** para os modos **Smart Web Flow**, **Web Flow** e **Smart Locators**.

Durante a gravação, o QANode Recorder tenta capturar:

- O elemento de origem antes do movimento começar
- O destino do drop, quando houver um elemento identificável
- As coordenadas de origem e destino como fallback
- Metadados do gesto para reproduzir o movimento com mais precisão

No modo **Smart Web Flow**, a gravação prioriza uma combinação de identidade do item arrastado, identidade do destino, contexto visual/estrutural e, quando necessário, coordenadas de apoio. Isso ajuda em quadros, kanbans, listas reordenáveis e áreas de drop que mudam depois do movimento.

No modo **Web Flow**, a gravação prioriza seletores CSS/XPath e atributos estáveis como `id`, `data-testid` e `data-qa`.

No modo **Smart Locators**, a extensão prioriza localizadores semânticos. Quando a página não fornece semântica suficiente, por exemplo imagens sem `alt`, áreas de drop sem texto ou botões apenas com ícone, a gravação pode usar um fallback técnico estável como `css: #id`. Isso evita localizadores frágeis como `getByRole("button")` sem nome ou dependência exclusiva de `nth`.

### Quando revisar o passo gravado

Revise manualmente o passo de drag/drop quando:

- O alvo for um canvas ou uma área livre sem elemento de destino claro
- Houver vários elementos com o mesmo texto
- O elemento for apenas um ícone sem `aria-label`
- A página mover/reordenar os elementos durante o gesto
- O destino for uma coluna vazia, uma lista virtualizada ou uma área que só aparece enquanto o mouse está arrastando

Para aplicações enterprise, prefira adicionar `data-testid`, `data-qa`, `aria-label` ou nomes acessíveis em botões e áreas de drop. Isso torna a gravação mais estável em todos os modos.

---

## Recursos Inteligentes

### Deduplicação

A extensão evita duplicação de passos:

- **Click + Navegação**: Se um click dispara uma navegação, o passo `navigate` extra é suprimido
- **Digitação consecutiva**: Preenchimentos consecutivos no mesmo campo são consolidados em um único passo
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
- Ao excluir um passo no popup, o estado interno da página também é sincronizado para que o passo removido não volte quando a gravação continuar

### Sessão do Navegador

Quando possível, a extensão também captura cookies e localStorage da página gravada junto com o JSON. Ao colar no QANode, essa sessão pode ser importada como storage persistido do nó, evitando que você precise refazer login antes da primeira execução.

Esse recurso é útil em ambientes autenticados como CRMs, ERPs e portais internos. Ainda assim, revise a sessão importada quando:

- a autenticação expira rápido
- o login depende de IP, MFA ou dispositivo
- o teste precisa começar obrigatoriamente sem cookies
- o fluxo será compartilhado com outro ambiente ou usuário

### Detecção de Navegação

A extensão usa o Performance API para distinguir entre:
- **Navegação** (nova URL) → gera passo `navigate`
- **Refresh** (mesma URL) → gera passo `refresh`
- **Click → Navegação** (link clicado) → suprime o `navigate` redundante
- **Click → Nova aba** → pode gerar `switchPage` para continuar a execução na aba aberta

### Frames e iFrames

Quando a ação acontece dentro de um iframe acessível, a extensão tenta gravar o contexto do frame antes da ação. Quando disponível, ela usa informações como seletor, nome e URL do frame para que a execução consiga voltar ao mesmo contexto.

Essa gravação melhora a confiabilidade em páginas com widgets, portais embedados e sistemas que carregam formulários dentro de iframes. Em iframes cross-origin ou muito dinâmicos, ainda pode ser necessário revisar o passo `frame` no QANode.

---

## Formato de Exportação

O JSON copiado é compatível com o nó correspondente ao modo selecionado.

### Smart Web Flow → Nó Smart Web Flow

```json
{
  "type": "smart-web-flow",
  "position": { "x": 100, "y": 100 },
  "data": {
    "label": "Recorded Smart Web Flow",
    "sessionMode": "new",
    "headless": true,
    "storageStrategy": "inMemory",
    "selfHealingEnabled": true,
    "smartWebSteps": [
      {
        "id": "swf-1",
        "action": "navigate",
        "label": "Navigate to page",
        "config": {
          "url": "https://exemplo.com/login",
          "expectedEffects": []
        },
        "evidence": {
          "takeScreenshot": true,
          "screenshotMode": "after"
        }
      },
      {
        "id": "swf-2",
        "action": "fill",
        "label": "Fill \"user@email.com\"",
        "config": {
          "locator": {
            "method": "getByLabel",
            "value": "E-mail",
            "exact": true
          },
          "selectorStrategies": [
            { "type": "css", "value": "input[name='email']" }
          ],
          "text": "user@email.com",
          "clearFirst": false
        }
      },
      {
        "id": "swf-3",
        "action": "click",
        "label": "Click \"Entrar\"",
        "config": {
          "locator": {
            "method": "getByRole",
            "value": "button",
            "name": "Entrar",
            "exact": true
          },
          "expectedEffects": [
            { "type": "domStable", "quietMs": 700, "timeoutMs": 15000 }
          ]
        }
      }
    ]
  }
}
```

> **Nota:** O Smart Web Flow pode incluir campos adicionais de identidade, contexto e efeitos esperados. Eles ajudam a execução. Pela UI, normalmente você só liga ou desliga o uso dos efeitos esperados do passo.

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

> **Diferenças principais:** No modo Smart Web Flow, o JSON usa `smartWebSteps` e pode carregar localizador, seletores alternativos, identidade do alvo, contexto e efeitos esperados. No modo Smart Locators, a ação `type` é convertida para `fill`, `select` para `selectOption`, e os seletores CSS são substituídos por localizadores semânticos (`locator` em vez de `selectorStrategies`).

---

## Fluxo de Trabalho Recomendado

```
1. Escolha o modo do recorder (preferencialmente Smart Web Flow para fluxos novos)
2. Grave a interação com a extensão
3. Cole no QANode (Ctrl+V)
4. Revise os passos gerados
5. Revise a chave de efeitos esperados e o texto de escopo quando existirem
6. Adicione waits onde necessário
7. Adicione asserts para validações (ou grave com Ctrl+A)
8. Ajuste seletores/localizadores se necessário
9. Configure screenshots nos passos críticos
10. Salve e execute
```

---

## Limitações

- **Apenas Chrome** — não funciona em outros navegadores
- **Nem toda espera é automática** — revise waits em páginas lentas, grids e carregamentos por API
- **Sites com CSP restrito** — podem bloquear a injeção de content script
- **iFrames cross-origin** — podem não ser acessíveis

---

## Dicas

- **Escolha o modo certo** — use Smart Web Flow para fluxos novos, Smart Locators para fluxos semânticos legados e Web Flow para seletores CSS/XPath tradicionais
- **Grave ações simples** e refine no QANode
- **Use Ctrl+A** (assert) durante a gravação em pontos de verificação
- **Use Ctrl+E** (extract) para capturar um valor único de um elemento
- **Use Ctrl+Shift+E** (extract list) para capturar dados de elementos repetidos como tabelas e listas de cards
- **Use Ctrl+Alt+W** (wait) quando um elemento precisa aparecer antes do próximo passo
- **Use Ctrl+Alt+T** (extract table) para tabelas e grids no modo Smart Web Flow
- **Adicione waits** no QANode para elementos que demoram a carregar
- **Revise o Texto do escopo** — em linhas, cards e listas, confirme se o texto identifica o item correto
- **Revise os seletores** — no modo Web Flow, prefira `data-testid`; nos modos semânticos, prefira `getByRole` ou `getByLabel`
- **Grave em segmentos** — para fluxos longos, grave por partes e combine no QANode
