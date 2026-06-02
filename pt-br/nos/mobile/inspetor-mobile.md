# Inspetor Mobile

O **Inspetor Mobile** é uma ferramenta visual integrada ao QANode para gravar e construir passos de automação mobile interativamente. Em vez de escrever XPath e seletores manualmente, você interage diretamente com a tela do dispositivo e os passos são gerados automaticamente.

---

## Como Acessar

O Inspetor Mobile é aberto a partir do editor de fluxos:

1. Adicione um nó **Mobile Flow** ao fluxo
2. Configure a credencial mobile (dispositivo, plataforma, app)
3. Clique no botão **"Lupa"** no painel de propriedades do nó

> Na **versão desktop**, o Appium é iniciado automaticamente ao conectar. Na versão web/server, certifique-se de que o Appium está rodando antes de abrir o inspetor.

---

## Interface

O Inspetor Mobile é dividido em três áreas principais:

```
┌──────────────────────────────────────────────────────────────┐
│  [Conectar]  Modo: [Toque | Inspecionar]   [Fechar Sessão]   │
├─────────────────────────┬────────────────────────────────────┤
│                         │  Ações Gravadas                    │
│   Tela do Dispositivo   │  ─────────────────────────────     │
│   (screenshot ao vivo)  │  1. tap — Botão Login              │
│                         │  2. type — email@...               │
│                         │  3. tap — Entrar                   │
│                         │  ─────────────────────────────     │
│                         │  [Limpar]          [Salvar Passos] │
├─────────────────────────┴────────────────────────────────────┤
│  Elemento Selecionado: id=com.app:id/login_btn               │
└──────────────────────────────────────────────────────────────┘
```

### Área da Tela do Dispositivo

Exibe o screenshot atualizado do dispositivo em tempo real. Você pode:
- **Tocar** em elementos para gerar passos `tap`
- **Arrastar** para gerar passos `swipe`
- **Clicar com clique-direito** ou modo Inspecionar para ver detalhes do elemento

### Área de Ações Gravadas

Lista todos os passos registrados durante a sessão. Cada ação mostra:
- Ícone e cor da ação, seguindo o padrão visual do QANode
- Tipo da ação (tap, type, swipe, wait, extract, assert, etc.)
- Descrição do alvo ou texto
- Ações para copiar ou remover o passo individualmente

Também é possível reorganizar passos arrastando-os na lista antes de salvar no nó.

### Painel do Elemento Selecionado

Exibe as informações do elemento tocado no modo **Inspecionar**:
- resumo do alvo selecionado;
- texto, label, classe nativa e bounds;
- atributos relevantes do elemento;
- seletores detectados, com opção de marcar ou desmarcar quais serão usados;
- quantidade de matches quando essa informação estiver disponível.

A seção de seletores começa recolhida para manter a tela limpa. Abra quando precisar revisar ou ajustar a estratégia antes de adicionar uma ação.

---

## Modos de Operação

### Modo Toque (padrão)

No modo **Toque**, cada clique na tela do dispositivo:
1. Envia o toque para o dispositivo via Appium
2. Grava um passo `tap` ou `tap-coords` com os seletores do elemento tocado
3. Atualiza o screenshot automaticamente após a ação

Para **deslizar** (swipe): clique e arraste na tela do dispositivo — o gesto é executado no dispositivo e gravado como passo `swipe`.

Quando um campo de texto está selecionado, o inspetor também permite gravar preenchimento com diálogo próprio, sem precisar alternar para outro modo. A opção **Limpar primeiro** vem marcada por padrão.

### Modo Inspecionar

No modo **Inspecionar**, clicar em um elemento **não envia o toque ao dispositivo** — apenas exibe as informações do elemento no painel inferior. Use este modo para:
- Explorar a hierarquia de elementos sem interagir com o app
- Copiar seletores para uso manual
- Identificar o melhor seletor antes de criar um passo

---

## Gravando Passos

### Passo de Toque (tap)

1. Certifique-se de estar no **Modo Toque**
2. Clique no elemento desejado na tela do dispositivo
3. O inspetor detecta o elemento, extrai seus seletores e grava um passo `tap`
4. A tela é atualizada automaticamente

**Atalho de duplo toque:** segure `Shift` e clique no elemento para executar e gravar `double-tap`.

### Passo de Digitação (type)

Após tocar em um campo de texto:
1. O inspetor detecta que o campo está ativo
2. Digite o texto no seu teclado físico — o inspetor captura as teclas em tempo real
3. O texto digitado é agrupado e gravado como passo `type`
4. Pressionar **Backspace** apaga caracteres e atualiza o passo

### Passo de Swipe

1. No **Modo Toque**, clique e **arraste** na tela do dispositivo
2. O gesto é executado no dispositivo
3. Um passo `swipe` é gravado com as coordenadas de início e fim

---

## Feedback Visual

Durante o uso do inspetor, indicadores visuais são exibidos sobre a tela do dispositivo:

| Indicador | Significado |
|-----------|-------------|
| Círculo verde pulsante | Ponto de toque registrado |
| Linha verde com círculos | Gesto de swipe registrado |

Esses indicadores desaparecem automaticamente após alguns segundos.

---

## Ações Avançadas

Além de gravar toques e swipes, o Inspetor Mobile oferece botões para adicionar ações avançadas diretamente ao fluxo sem precisar configurar manualmente os campos.

### Ações disponíveis no painel

**Com elemento selecionado (modo Inspecionar):**

| Ação | Descrição |
|------|-----------|
| **Toque** | Cria um passo `tap` no elemento selecionado |
| **Preencher** | Abre um diálogo para informar o texto e cria um passo `type` |
| **Duplo Toque** | Cria um passo de duplo toque no elemento selecionado |
| **Pressionar e Segurar** | Cria um passo `long-press` no elemento selecionado |
| **Pinça para Dentro** | Cria um gesto de pinça sobre o elemento |
| **Zoom para Fora** | Cria um gesto de afastamento sobre o elemento |
| **Esperar Visível** | Cria uma espera até o elemento aparecer |
| **Esperar Habilitado** | Cria uma espera até o elemento estar habilitado |
| **Extração** | Cria um passo `extract` usando seletores estáveis |
| **Validação** | Cria um passo de assert para o elemento selecionado |

**Ações de sistema (sem elemento selecionado):**

| Ação | Descrição |
|------|-----------|
| **Enviar arquivo** | Envia um arquivo para o dispositivo e cria um passo `push-file` |
| **Baixar arquivo** | Captura um arquivo do dispositivo e cria um passo `pull-file` |
| **Aceitar Alerta** | Cria um passo `permission` que aceita o alerta de sistema atual |
| **Dispensar Alerta** | Cria um passo `permission` que dispensa o alerta de sistema atual |
| **Conceder Permissão** | Solicita o nome da permissão Android e cria um passo para concedê-la |
| **Revogar Permissão** | Solicita o nome da permissão Android e cria um passo para revogá-la |

> As ações **Conceder Permissão** e **Revogar Permissão** são exclusivas para sessões Android e abrem um prompt solicitando o nome da permissão, por exemplo: `android.permission.CAMERA`.

### Enviar arquivo durante a gravação

Use **Enviar arquivo** quando o app precisa receber um arquivo antes de abrir um seletor nativo ou continuar o fluxo. O inspetor envia o arquivo para o dispositivo durante a gravação e adiciona o passo correspondente ao Mobile Flow.

No Android, o destino padrão costuma ser a pasta de downloads. No iOS, o inspetor usa o container do aplicativo quando disponível e aplica fallback compatível com o Appium.

### Baixar arquivo durante a gravação

Use **Baixar arquivo** quando o app gerou um arquivo e o teste precisa capturá-lo como `fileRef`.

O diálogo permite informar:

- caminho exato no dispositivo;
- pasta e padrão de nome para buscar o arquivo mais recente;
- nome da variável de saída;
- nome do arquivo salvo no QANode.

Use caminho exato quando o app mostra ou define o nome do arquivo. Use pasta/padrão quando o nome é dinâmico.

---

## Salvando os Passos

Ao clicar em **"Salvar Passos"**:
1. Todos os passos gravados na sessão são transferidos para o nó Mobile Flow
2. A sessão Appium permanece aberta (pode continuar gravando depois se necessário)
3. O inspetor é fechado e os passos ficam visíveis no painel do nó

> Os passos gerados pelo inspetor podem ser editados manualmente no nó Mobile Flow após salvar — adicionar evidências, ajustar seletores, configurar tentativas, etc.

---

## Gerenciando a Sessão

| Ação | Descrição |
|------|-----------|
| **Conectar** | Inicia uma nova sessão Appium com as configurações da credencial |
| **Fechar Sessão** | Encerra a sessão Appium no dispositivo |
| **Limpar Passos** | Remove todos os passos gravados (a sessão continua ativa) |

> Fechar o inspetor sem clicar em "Salvar Passos" descarta os passos gravados na sessão atual. Passos já salvos anteriormente no nó não são afetados.

---

## Seletores Gerados

O inspetor gera automaticamente seletores para cada elemento tocado, priorizando nesta ordem:

1. **`accessibility_id`**, `id`, `resource-id`, `name` ou `content-desc` — sinais mais estáveis
2. **`hint`**, `label` e `text` quando pertencem ao próprio elemento
3. **Contexto próximo** — label/texto associado ao campo, útil para inputs sem identificador
4. **`xpath`** com `@bounds` — seletor baseado nas coordenadas do elemento
5. **Coordenadas de fallback** — X/Y absolutos usados se todos os seletores falharem
6. **Classe genérica** — usada apenas como último recurso

> Os seletores com `@bounds` e as coordenadas são fallback visual. Em fluxos automatizados, prefira seletores por `accessibility_id`, `id`, `resource-id`, `label` ou contexto próximo.

---

## Dicas

- Use o **Modo Inspecionar** para identificar o melhor seletor antes de gravar, especialmente em telas com muitos elementos semelhantes
- **Grave um fluxo completo** de uma vez — conectar, executar o caso de uso no app, salvar — para obter passos com contexto real
- Após salvar, revise os passos no nó e adicione **screenshots de evidência** nos passos críticos
- Para apps com animações longas, adicione passos `wait` (elementVisible) após os toques para garantir que a tela carregou antes da próxima ação
- Na **versão desktop**, a conexão com o dispositivo local é mais rápida — use para desenvolvimento. Para testes em cloud (BrowserStack, etc.), configure a credencial SaaS antes de abrir o inspetor
