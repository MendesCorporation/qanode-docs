# Nó Mobile Flow

O nó **Mobile Flow** permite automatizar interações com aplicativos móveis nativos (Android e iOS) usando o **Appium**. Suporta dispositivos reais, emuladores e serviços cloud como BrowserStack, Sauce Labs e LambdaTest.

---

## Visão Geral

[Visão Geral](../../assets/images/nos-mobile-flow-visao-geral.mp4)

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `mobile-flow` |
| **Categoria** | Mobile |
| **Cor** | 🔴 Vermelho claro (#f87171) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Pré-requisitos

### Appium

O nó Mobile Flow requer um servidor **Appium** acessível:

| Situação | Solução |
|----------|---------|
| **Versão Desktop** | O Appium é instalado automaticamente no primeiro uso e iniciado ao executar um fluxo com nó mobile |
| **Servidor próprio** | Instale manualmente: `npm install -g appium` + driver: `appium driver install uiautomator2` (Android) ou `appium driver install xcuitest` (iOS) |
| **Cloud (BrowserStack, etc.)** | Nenhum Appium local necessário — use as credenciais do provedor |

#### Versões recomendadas (QANode 0.2.2+)

```bash
npm install -g appium@3.2.0
appium driver install --source=npm appium-uiautomator2-driver@7.0.0
appium driver install --source=npm appium-xcuitest-driver@10.25.0
```

> Para iOS, o XCUITest requer macOS com Xcode configurado.

### Drivers Appium

| Plataforma | Driver | Instalação |
|-----------|--------|------------|
| Android | UiAutomator2 | `appium driver install uiautomator2` |
| iOS | XCUITest | `appium driver install xcuitest` (requer Xcode no Mac) |

---

## Configuração Geral

| Campo | Tipo | Padrão | Descrição |
|-------|------|--------|-----------|
| **Credencial** | `select` | — | Credencial Mobile com configurações de conexão |
| **Modo de Sessão** | `new` / `reuse` | `new` | Nova sessão ou reutilizar sessão existente |
| **Session ID** | `string` | — | ID para identificar/reutilizar a sessão |
| **Self Healing** | `boolean` | `false` | Tenta recuperar elementos quando os seletores mobile deixam de localizar o alvo original |

### Modo de Sessão

- **Nova Sessão (`new`)**: Abre uma nova sessão Appium a cada execução. Ideal para testes isolados.
- **Reutilizar Sessão (`reuse`)**: Reaproveita uma sessão já criada por outro nó Mobile Flow no mesmo fluxo. Útil para dividir automações longas em múltiplos nós mantendo o mesmo dispositivo conectado.

### Self Healing

Quando **Self Healing** está habilitado, o Mobile Flow tenta recuperar automaticamente elementos em ações como toque e digitação quando os seletores gravados deixam de funcionar.

No mobile, a recuperação considera sinais como:

- `accessibility_id`
- texto visível
- `label`, `name`, `value` e `hint`
- `resource-id`
- tipo/classe nativa do elemento
- contexto próximo na hierarquia capturada pelo Appium

Isso ajuda em mudanças comuns como:

- ajustes pequenos de rótulo
- diferenças de maiúsculas/minúsculas
- texto com ou sem acento
- mudança de seletor mantendo o mesmo elemento na tela

O self healing em mobile também respeita a plataforma ativa:

- prioriza estratégias compatíveis com iOS quando a execução está em iOS
- prioriza estratégias compatíveis com Android quando a execução está em Android

Quando uma recuperação é aplicada, ela aparece nos detalhes da execução com:

- passo afetado
- confiança da recuperação
- seletor mobile utilizado
- ação **Aplicar ao Fluxo** para promover o seletor recuperado ao fluxo original

---

## Credencial Mobile

A credencial centraliza as configurações de conexão e capacidades do dispositivo. Configure em **Credenciais → Nova Credencial → Mobile**.

### Modos de Conexão

| Modo | Descrição |
|------|-----------|
| **Local** | Appium rodando na mesma máquina (padrão: `http://localhost:4723`) |
| **Self-hosted** | Appium em servidor próprio com URL e token de autenticação |
| **SaaS** | Provedor cloud (BrowserStack, Sauce Labs, LambdaTest, ou Custom) |

### Capacidades do Dispositivo

| Campo | Descrição | Exemplo |
|-------|-----------|---------|
| **Plataforma** | `Android` ou `iOS` | `Android` |
| **Nome do Dispositivo** | Nome ou ID do dispositivo | `emulator-5554`, `Pixel 7` |
| **Versão da Plataforma** | Versão do SO | `13.0`, `17.0` |
| **App Package** | Pacote do app Android | `com.meuapp.android` |
| **App Activity** | Activity inicial do Android | `.MainActivity` |
| **Bundle ID** | Identificador do app iOS | `com.meuapp.ios` |
| **App Path** | Caminho do arquivo .apk/.ipa | `/path/to/app.apk` |
| **UDID** | ID único do dispositivo real | `R58M20XXXXX` |
| **No Reset** | Não resetar o app entre sessões | `true` |
| **Full Reset** | Resetar completamente o estado do app | `false` |
| **Automation Name** | Driver de automação | `UiAutomator2` (Android), `XCUITest` (iOS) |
| **Capacidades Extras** | JSON com capacidades adicionais do Appium | `{"appium:language": "pt"}` |

---

## Passos (Mobile Steps)

O nó Mobile Flow executa uma sequência de **passos** configuráveis. Cada passo representa uma ação no dispositivo.

Quando o fluxo é gravado via MCP, os passos aparecem no editor visual com o mesmo padrão do Inspetor Mobile, incluindo destaques de interação e informações úteis para revisar falhas.

Use o MCP para gravar ou ajustar jornadas mobile quando a IA precisa interagir com Appium, mas revise no editor visual se o alvo gravado representa a intenção correta.

### Ações Disponíveis

| Ação | Descrição |
|------|-----------|
| [tap](#tap) | Tocar em um elemento |
| [double-tap](#double-tap) | Toque duplo em um elemento |
| [long-press](#long-press) | Pressionar e segurar um elemento |
| [tap-coords](#tap-coords) | Tocar em coordenadas absolutas |
| [type](#type) | Digitar texto em um campo |
| [clear](#clear) | Limpar texto de um campo |
| [swipe](#swipe) | Deslizar na tela |
| [pinch-zoom](#pinch-zoom) | Gesto de pinça / zoom |
| [multi-touch](#multi-touch) | Toque simultâneo em múltiplos pontos |
| [scroll](#scroll) | Rolar por direção ou até elemento/texto |
| [wait](#wait) | Aguardar condição ou tempo |
| [assert](#assert) | Verificar condição no app |
| [extract](#extract) | Extrair texto ou atributo de elemento |
| [permission](#permission) | Aceitar/dispensar alertas e gerenciar permissões |
| [push-file](#push-file) | Enviar arquivo para o dispositivo |
| [pull-file](#pull-file) | Baixar arquivo do dispositivo para o QANode |
| [reset](#reset) | Reiniciar o aplicativo |
| [back](#back) | Pressionar botão Voltar (Android) |
| [home](#home) | Pressionar botão Home (Android) |
| [key-event](#key-event) | Enviar código de tecla Android |

---

## Estratégias de Seletor Mobile

| Estratégia | Descrição | Exemplo |
|------------|-----------|---------|
| `id` | Resource ID do elemento | `com.meuapp:id/login_button` |
| `accessibility_id` | Content description / accessibility label | `Botão de login` |
| `xpath` | Expressão XPath na hierarquia XML | `//android.widget.Button[@text="Entrar"]` |
| `class_name` | Nome da classe nativa | `android.widget.EditText` |
| `-android uiautomator` | Seletor UIAutomator2 (Android) | `new UiSelector().text("Entrar")` |
| `-ios predicate string` | NSPredicate (iOS) | `type == 'XCUIElementTypeButton' AND name == 'Entrar'` |
| `-ios class chain` | iOS Class Chain | `**/XCUIElementTypeButton[\`name == 'Entrar'\`]` |

> **Dica:** O [Inspetor Mobile](inspetor-mobile.md) gera automaticamente os seletores ao tocar nos elementos, sem necessidade de escrever XPath manualmente.

---

## Detalhes das Ações

### tap

Toca em um elemento usando estratégias de seletor.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Seletores** | `array` | Estratégias de seletor do elemento |
| **Tentativas** | `number` | Número de tentativas (padrão: 3) |
| **Delay entre tentativas (ms)** | `number` | Espera entre tentativas (padrão: 500ms) |

Caso o seletor falhe em todas as tentativas e houver coordenadas de fallback (geradas pelo inspetor), o toque será realizado nas coordenadas diretas.

---

### double-tap

Toque duplo em um elemento.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Seletores** | `array` | Estratégias de seletor do elemento |
| **Tentativas** | `number` | Número de tentativas (padrão: 3) |

---

### long-press

Pressiona e segura um elemento por uma duração determinada. Útil para abrir menus de contexto, acionar seleção de texto e outros gestos longos.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Seletores** | `array` | Estratégias de seletor do elemento (opcional se houver X/Y) |
| **X / Y** | `number` | Coordenadas de fallback quando não há seletor |
| **Duração (ms)** | `number` | Tempo de pressionamento (padrão: 800ms) |

---

### tap-coords

Toca em coordenadas absolutas na tela.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **X** | `number` | Coordenada X em pixels |
| **Y** | `number` | Coordenada Y em pixels |

---

### type

Digita texto em um campo de entrada.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Seletores** | `array` | Estratégias de seletor do campo |
| **Texto** | `string` | Texto a ser digitado (suporta `{{ }}`) |
| **Limpar Primeiro** | `boolean` | Limpa o campo antes de digitar |
| **Tentativas** | `number` | Número de tentativas de localização |

Para campos de texto, o QANode tenta localizar em camadas:

1. identificadores fortes, como resource-id, accessibility id, name e content-desc;
2. `hint`, `label` e `text` quando pertencem ao próprio campo;
3. relação contextual entre o label visível e o input próximo;
4. coordenadas gravadas como fallback visual;
5. classe genérica de input apenas como último recurso.

---

### clear

Limpa o conteúdo de um campo de entrada.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Seletores** | `array` | Estratégias de seletor do campo |

---

### swipe

Desliza na tela entre dois pontos.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **fromX / fromY** | `number` | Coordenadas de início |
| **toX / toY** | `number` | Coordenadas de destino |
| **Duração (ms)** | `number` | Velocidade do gesto (padrão: 350ms) |
| **Direção** | `string` | Alternativa: `up`, `down`, `left`, `right` |
| **Percentual** | `number` | Percentual da tela para deslizar (com direção) |

---

### pinch-zoom

Executa um gesto de pinça (aproximar dedos) ou zoom (afastar dedos) sobre um elemento ou ponto da tela.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Gesto** | `string` | `pinchIn` (pinça, reduz) ou `zoomOut` (zoom, amplia) |
| **Seletores** | `array` | Seletor do elemento central do gesto (opcional) |
| **X / Y** | `number` | Coordenadas centrais de fallback |
| **Distância** | `number` | Distância em pixels entre os dois dedos (padrão: 160) |
| **Escala** | `number` | Fator de escala do gesto (padrão: 0.45) |
| **Duração (ms)** | `number` | Velocidade do gesto (padrão: 500ms) |

---

### multi-touch

Simula um toque simultâneo em dois ou mais pontos da tela.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Pontos** | `array` | Lista de coordenadas `{ x, y }` a tocar simultaneamente |
| **X1 / Y1** | `number` | Primeiro ponto (alternativa a `points`) |
| **X2 / Y2** | `number` | Segundo ponto (alternativa a `points`) |
| **Duração (ms)** | `number` | Duração do toque (padrão: 120ms) |

---

### scroll

Rola o conteúdo em uma direção ou continua rolando até que um elemento ou texto apareça na tela.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Modo** | `string` | `direction` (padrão), `untilElement` ou `untilText` |
| **Direção** | `string` | `up`, `down`, `left`, `right` |
| **Percentual** | `number` | Percentual da tela a rolar por ciclo (padrão: 70) |
| **Seletores** | `array` | Seletor do container a rolar, ou do elemento-alvo nos modos `untilElement` |
| **Texto** | `string` | Texto a aguardar na tela (modo `untilText`) |
| **Máximo de Rolagens** | `number` | Limite de ciclos de rolagem nos modos `until*` (padrão: 8) |

**Modos:**

| Modo | Descrição |
|------|-----------|
| `direction` | Rola uma vez na direção configurada |
| `untilElement` | Rola repetidamente até o elemento dos seletores estar visível |
| `untilText` | Rola repetidamente até o texto aparecer no XML da tela |

---

### wait

Aguarda uma condição antes de prosseguir.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Modo** | `string` | Tipo de espera |
| **Timeout (ms)** | `number` | Tempo máximo de espera |
| **Seletores** | `array` | Seletores do elemento alvo (quando aplicável) |
| **Texto Esperado** | `string` | Valor de referência (modos `textContains`, `textEquals`) |
| **Atributo** | `string` | Nome do atributo (modo `attributeEquals`) |
| **Valor Esperado** | `string` | Valor do atributo (modo `attributeEquals`) |

**Modos:**

| Modo | Descrição |
|------|-----------|
| `timeout` | Aguarda um tempo fixo em milissegundos |
| `elementVisible` | Aguarda um elemento aparecer na tela |
| `elementHidden` | Aguarda um elemento desaparecer ou ficar oculto |
| `elementEnabled` | Aguarda um elemento ficar habilitado para interação |
| `textContains` | Aguarda o texto do elemento conter o valor esperado |
| `textEquals` | Aguarda o texto do elemento ser exatamente o valor esperado |
| `attributeEquals` | Aguarda um atributo do elemento ter o valor esperado |
| `screenChanged` | Aguarda a tela mudar visualmente (screenshot diferente do inicial) |

---

### assert

Verifica uma condição no aplicativo. Se falhar, o passo é marcado como falha.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Nome** | `string` | Identificador da asserção (chave no output) |
| **Modo** | `string` | Tipo de verificação |
| **Seletores** | `array` | Seletores do elemento |
| **Texto Esperado** | `string` | Valor esperado (modos text) |
| **Continuar em Falha** | `boolean` | Não interrompe o fluxo se falhar |

**Modos:**

| Modo | Descrição |
|------|-----------|
| `elementExists` | Verifica se o elemento está visível na tela |
| `textContains` | Verifica se o texto do elemento contém o valor esperado |
| `textEquals` | Verifica se o texto do elemento é exatamente o valor esperado |

Os resultados das asserções ficam disponíveis nos outputs:
```
{{ steps["mobile-flow"].outputs.asserts.nomeAsserção }}  →  true ou false
```

---

### extract

Extrai texto ou atributo de um elemento.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Nome** | `string` | Nome da extração (chave no output) |
| **Seletores** | `array` | Seletores do elemento |
| **Atributo** | `string` | O que extrair: `text` (padrão) ou nome do atributo |

Os dados extraídos ficam disponíveis nos outputs:
```
{{ steps["mobile-flow"].outputs.extracts.nomeExtracao }}
```

---

### permission

Gerencia alertas de sistema e permissões do aplicativo.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Operação** | `string` | Tipo de ação (ver tabela abaixo) |
| **Permissão** | `string` | Nome da permissão Android (somente `grantPermission`/`revokePermission`) |

**Operações:**

| Operação | Descrição |
|----------|-----------|
| `acceptAlert` | Aceita o alerta de sistema atual (ex: "Permitir notificações?") |
| `dismissAlert` | Dispensa o alerta de sistema atual |
| `grantPermission` | Concede uma permissão Android ao app (ex: `android.permission.CAMERA`) |
| `revokePermission` | Revoga uma permissão Android do app |

> `grantPermission` e `revokePermission` são suportados apenas em sessões Android com o UiAutomator2.

---

### push-file

Envia um arquivo do QANode para o dispositivo antes de continuar a automação.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Arquivo** | `fileRef` | Arquivo a enviar |
| **Caminho no dispositivo** | `string` | Caminho de destino. Se vazio, o QANode usa um local padrão da plataforma |

Use este passo quando o aplicativo precisa que um arquivo exista no dispositivo antes de abrir um seletor nativo, galeria, picker de documentos ou fluxo de upload.

No Android, caminhos comuns ficam em `/sdcard/Download/...`. No iOS, o QANode tenta enviar para o container do aplicativo quando disponível e usa fallback compatível com o Appium/XCUITest.

---

### pull-file

Baixa um arquivo do dispositivo e salva como artefato do QANode, retornando `fileRef`.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Modo** | seleção | Caminho exato, pasta/padrão ou detecção mais recente |
| **Caminho no dispositivo** | `string` | Caminho exato do arquivo, quando conhecido |
| **Pasta** | `string` | Pasta a consultar, como `/sdcard/Download` |
| **Padrão** | `string` | Filtro de nome, como `*.pdf`, `*.txt`, `*.jpg` ou `*` |
| **Nome da variável** | `string` | Chave em `outputs.files` |
| **Nome do arquivo de saída** | `string` | Nome salvo no QANode. Se vazio, usa o nome encontrado |

Quando possível, prefira **Caminho exato**. Quando o aplicativo gera o arquivo com nome dinâmico, use pasta/padrão para capturar o arquivo mais recente compatível.

Arquivos como imagens, PDFs, textos, planilhas e vídeos podem ser capturados desde que o Appium consiga ler o arquivo no dispositivo ou no container do aplicativo.

---

### reset

Reinicia o aplicativo, limpando seu estado (equivalente a desinstalar e reinstalar).

---

### back

Pressiona o botão físico/virtual **Voltar** (Android — keycode 4).

---

### home

Pressiona o botão **Home** (Android — keycode 3).

---

### key-event

Envia um keycode Android diretamente ao dispositivo.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Keycode** | `number` | Código da tecla Android (ex: 4 = BACK, 3 = HOME, 66 = ENTER) |
| **Repetições** | `number` | Quantas vezes enviar o evento |

**Keycodes comuns:**

| Keycode | Tecla |
|---------|-------|
| 3 | HOME |
| 4 | BACK |
| 66 | ENTER |
| 67 | BACKSPACE/DEL |
| 82 | MENU |
| 84 | SEARCH |

---

## Evidências (Screenshots)

Cada passo pode ter configuração de evidência individual:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Capturar Screenshot** | `boolean` | Ativar captura de tela |
| **Modo** | `before` / `after` / `both` | Quando capturar |
| **Template de Nome** | `string` | Nome customizado do arquivo |
| **Aguardar Estabilização** | `boolean` | Esperar tela estável antes de capturar |
| **Timeout de Estabilização (ms)** | `number` | Tempo máximo aguardando tela estável (padrão: 2000ms) |
| **Delay (ms)** | `number` | Tempo adicional antes de capturar (padrão: 300ms) |

> Os screenshots gerados ficam disponíveis na aba **Evidências** do resultado da execução. Toques e swipes são destacados com marcação visual verde sobre o screenshot.

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `sessionId` | `string` | ID da sessão mobile para reutilização em outros nós |
| `extracts` | `object` | Dados extraídos pelos passos `extract` (chave → valor) |
| `asserts` | `object` | Resultados das asserções (chave → `true`/`false`) |
| `files` | `object` | Arquivos baixados por passos `pull-file` |
| `fileRef` | `fileRef` | Atalho para o último arquivo baixado |

### Arquivos baixados

Quando um passo **Baixar arquivo** é executado, o output fica disponível pelo nome da variável:

```
{{ steps["mobile-flow"].outputs.files.relatorio.fileRef }}
{{ steps["mobile-flow"].outputs.files.relatorio.name }}
{{ steps["mobile-flow"].outputs.fileRef }}
```

Use o `fileRef` para passar o arquivo para **Extrair Arquivo**, **Arquivo para Base64**, **HTTP Request**, componentes ou outros nós que aceitam arquivo.

---

## Exemplo Completo

### Login em aplicativo Android

Passos configurados no nó:

1. **wait** → Modo: `elementVisible`, Seletor: `id=com.meuapp:id/email_field` (timeout: 10000ms)
2. **tap** → Seletor: `id=com.meuapp:id/email_field`
3. **type** → Seletor: `id=com.meuapp:id/email_field`, Texto: `{{ variables.USER_EMAIL }}`
4. **type** → Seletor: `id=com.meuapp:id/password_field`, Texto: `{{ variables.USER_PASSWORD }}`, Limpar: true
5. **tap** → Seletor: `accessibility_id=Entrar`
6. **wait** → Modo: `elementVisible`, Seletor: `id=com.meuapp:id/home_title` (timeout: 15000ms)
7. **assert** → Nome: `loginOk`, Modo: `elementExists`, Seletor: `id=com.meuapp:id/home_title`
8. **extract** → Nome: `welcomeText`, Seletor: `id=com.meuapp:id/welcome_message`, Atributo: `text`

**Resultado após execução:**
```json
{
  "sessionId": "minha-sessao-android",
  "extracts": { "welcomeText": "Olá, João!" },
  "asserts": { "loginOk": true }
}
```

---

### Reutilizando Sessão entre Nós

**Nó 1 — Mobile Flow (Login)**
- Session ID: `sessao-android`
- Passos: login no app

**Nó 2 — Mobile Flow (Verificar Pedido)**
- Modo de Sessão: `reuse`
- Session ID: `sessao-android`
- Passos: navegar para pedidos, verificar status

Ambos os nós compartilham o mesmo dispositivo e estado de sessão.

---

## Dicas

- Use o **[Inspetor Mobile](inspetor-mobile.md)** para gravar passos visualmente — ele gera os seletores automaticamente ao tocar nos elementos
- Na **versão desktop**, o Appium é iniciado automaticamente quando um fluxo com nó mobile é executado — não é necessário nenhuma configuração manual
- Prefira `accessibility_id` como seletor principal — é mais estável que XPath e funciona em Android e iOS
- Configure **screenshots** nos passos críticos para facilitar a identificação de falhas
- Use **Continuar em Falha** em asserções de monitoramento para coletar múltiplos resultados sem interromper o fluxo
- Para apps que demoram para iniciar, adicione um passo `wait` com `elementVisible` antes de interagir
- O `Session ID` personalizado facilita a reutilização de sessão entre múltiplos nós Mobile Flow no mesmo fluxo
