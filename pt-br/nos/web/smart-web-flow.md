# Nó Smart Web Flow

O nó **Smart Web Flow** é o nó recomendado para automação web no QANode. Ele combina gravação pela extensão Chrome, localizadores semânticos, seletores CSS/XPath, contexto visual da página, validações de efeito esperado e mecanismos conservadores de recuperação para executar cenários em aplicações web modernas.

Ele foi criado para lidar melhor com sistemas enterprise e páginas dinâmicas, como CRMs, ERPs, ServiceNow, Salesforce, SAP, Jira, portais internos, aplicações com iframes, menus, grids, kanban, componentes ricos e interfaces que mudam pequenas partes do DOM entre uma execução e outra.

> **Recomendação:** Para novos cenários web, prefira **Smart Web Flow**. Os nós [Web Flow](web-flow.md) e [Smart Locators](smart-locators.md) continuam disponíveis para compatibilidade e casos específicos.

---

## Visão Geral

[Visão Geral](../../assets/images/nos-smart-web-flow-visao-geral.mp4)

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `smart-web-flow` |
| **Categoria** | Web |
| **Cor** | 🔵 Azul (#3b82f6) |
| **Entrada** | `in` |
| **Saída** | `out` |
| **Gravação recomendada** | QANode Recorder no modo Smart Web Flow |
| **Localização de elementos** | Semântica + CSS/XPath + identidade do alvo + contexto |
| **Uso recomendado** | Fluxos web novos, aplicações dinâmicas, testes com evidências e recuperação assistida |

---

## Quando Usar

Use o **Smart Web Flow** quando:

- o fluxo será gravado pela extensão Chrome;
- a tela possui menus, dropdowns, modais, iframes, listas, tabelas, grids ou cards;
- os elementos podem mudar de posição entre execuções;
- você precisa de screenshots por passo;
- você quer validações automáticas após ações importantes;
- você quer receber sugestões de melhoria após execuções bem-sucedidas;
- o mesmo fluxo precisa rodar com dados variáveis, como número de chamado, nome de cliente, card, item ou registro.

Considere usar outro nó quando:

- você já possui um fluxo legado simples em Web Flow e ele está estável;
- você precisa de um seletor CSS/XPath manual muito específico e não quer usar os metadados do Smart Web;
- você está automatizando algo que não é web, como API, banco de dados ou mobile nativo.

---

## Configuração Geral

| Campo | Tipo | Padrão | Descrição |
|-------|------|--------|-----------|
| **Modo de Sessão** | `new` / `reuse` | `new` | Define se o nó abre uma nova sessão de navegador ou reutiliza uma sessão existente |
| **ID da Sessão** | `string` | — | Nome ou expressão usada para reutilizar a sessão |
| **Headless** | `boolean` | `true` | Executa o navegador sem interface visível |
| **Estratégia de Storage** | `inMemory` / `persisted` | `inMemory` | Controla se cookies/localStorage serão descartados ou persistidos |
| **ID da Sessão Persistida** | `string` | — | Nome opcional usado para salvar/reutilizar cookies e localStorage |
| **Browser** | `chromium` / `firefox` / `webkit` | `chromium` | Navegador usado na execução |
| **Dispositivo** | `desktop` / `mobile` | `desktop` | Define se a execução usa viewport desktop ou emulação mobile |
| **Viewport** | preset ou `custom` | `1920x1080` | Tamanho da janela em modo desktop |
| **Self Healing** | `boolean` | `true` | Permite recuperação conservadora quando o alvo original não é encontrado |
| **Varredura de acessibilidade** | `boolean` | `false` | Executa análise de acessibilidade e gera evidências |
| **Auditoria de performance** | `boolean` | `false` | Coleta métricas de navegação, rede e APIs por tela |

### Storage e Sessão

Para fluxos que exigem login, MFA ou SSO, use storage persistido. Assim, o QANode pode reutilizar cookies e localStorage em execuções futuras sem refazer o login manualmente.

Use `inMemory` para testes isolados e `persisted` para aplicações que precisam manter autenticação entre execuções.

Quando o modo de sessão é **novo** e o storage é **persistido**, o campo de ID da sessão persistida é opcional:

- se preenchido, salva a sessão com esse nome;
- se vazio, o QANode gera uma chave interna automaticamente.

Sessões persistidas criadas pelo nó ficam disponíveis para reutilização por outros usuários da mesma instalação. Isso permite que um fluxo compartilhado continue funcionando quando outra pessoa executa o cenário.

Sessões importadas pela extensão Chrome permanecem vinculadas ao usuário que gravou/importou a sessão, porque normalmente representam login, cookies e permissões pessoais.

No modo **reutilizar**, o campo **ID da Sessão** busca sessões nomeadas dos dois escopos. As sugestões mostram até 10 sessões e escondem chaves geradas automaticamente por UUID. Sessões sem uso por 30 dias são limpas automaticamente.

---

## Como o Smart Web Flow Encontra Elementos

Cada passo pode ter mais de uma pista para encontrar o alvo correto. Em vez de depender de um único seletor, o Smart Web Flow combina sinais diferentes:

| Sinal | Exemplo | Quando ajuda |
|------|---------|--------------|
| **Localizador semântico** | `getByRole`, `getByLabel`, `getByText` | Botões, links, campos, opções e elementos com acessibilidade razoável |
| **Seletores CSS/XPath** | `#email`, `[data-testid="submit"]`, XPath | Páginas com IDs, atributos estáveis ou estrutura conhecida |
| **Identidade do alvo** | texto, role, tag, atributos, posição aproximada | Confirma que o seletor encontrado ainda representa o mesmo elemento |
| **Contexto de linha/card** | referência do alvo, item pai, linha de tabela | Telas com elementos repetidos |
| **Contexto de controle** | label próximo, valor esperado, tipo do controle | Inputs customizados, combos, calendários e checkboxes estilizados |
| **Frame/Iframe** | contexto do frame gravado | Aplicações que renderizam conteúdo dentro de iframes |

O objetivo é ser resiliente sem clicar em qualquer coisa parecida. Quando a tela fica ambígua, o Smart Web Flow tende a falhar e mostrar diagnóstico em vez de escolher um alvo incerto.

---

## Referência do Alvo

O campo **Referência do alvo** aparece quando o passo foi gravado dentro de um item repetido, como:

- linha de tabela;
- card de kanban;
- item de lista;
- bloco de resultado;
- container com vários botões iguais.

Ele informa ao QANode **qual registro, linha, card ou item deve servir de referência para encontrar o alvo**.

Em termos práticos: primeiro o QANode localiza a referência, depois procura o botão, campo, texto ou ação dentro daquele contexto.

### Exemplo

Imagine uma tabela com vários botões **Abrir**:

| Chamado | Cliente | Ação |
|---------|---------|------|
| INC0010001 | Apptrix | Abrir |
| INC0010002 | QANode | Abrir |

Se o alvo é o botão **Abrir** da linha `INC0010002`, o localizador do botão sozinho não é suficiente. A **Referência do alvo** pode ser:

```
INC0010002
```

Assim, o QANode procura primeiro a linha/card que contém esse texto e só depois resolve o elemento dentro desse contexto.

### Usando Variáveis

A referência do alvo aceita expressões:

```
{{ variables.numeroChamado }}
{{ steps["Criar chamado"].outputs.extracts.numero }}
```

Isso é útil quando o dado muda a cada execução, mas a intenção permanece a mesma: agir no card, linha ou item correspondente ao valor daquele teste.

### Boas Práticas

- Use uma referência curta e única dentro do item, como número, código, nome ou título.
- Evite usar o texto completo de um card grande se uma parte menor já identifica o item.
- Para dados variáveis, use `{{ }}` em vez de deixar o valor gravado fixo.
- Se vários itens possuem textos parecidos, complemente o fluxo com asserts ou extrações para confirmar o item correto.
- Quando possível, peça ao time de desenvolvimento para expor `data-testid`, `data-qa`, labels ou nomes acessíveis.

---

## Estratégias e Ordem de Tentativa

Em uma execução normal, o Smart Web Flow tenta resolver o passo usando as estratégias gravadas e os metadados disponíveis. Em termos práticos:

1. tenta a estratégia preferida quando ela já foi validada em execuções anteriores;
2. tenta localizadores semânticos, quando aplicáveis;
3. tenta seletores CSS/XPath registrados;
4. usa contexto de linha/card/controle quando o alvo é repetido ou customizado;
5. se **Self Healing** estiver habilitado, tenta recuperação conservadora;
6. se não houver confiança suficiente, o passo falha com diagnóstico.

O usuário não precisa configurar essa ordem manualmente na maioria dos casos. O importante é revisar se o passo gravado representa a intenção correta: elemento, texto, escopo, ação e efeito esperado.

---

## Efeitos Esperados

Alguns passos gravados possuem **Efeitos esperados** gerados automaticamente pela extensão ou pelo motor. Eles descrevem o que o QANode observou como resultado provável de uma ação, por exemplo:

| Efeito | O que valida |
|--------|--------------|
| `domStable` | A página estabilizou por um curto período |
| `networkIdle` | A rede ficou ociosa dentro do limite configurado |
| `urlContains` | A URL passou a conter um trecho esperado |
| `titleContains` | O título da página passou a conter um trecho esperado |
| `elementVisible` | Um seletor específico ficou visível |
| `targetVisible` | O próximo alvo esperado ficou visível |
| `overlayOpened` | Um menu, dropdown, modal ou overlay abriu |
| `pageOpened` | Uma nova aba/janela foi aberta |

### Como Interpretar

Na UI, o usuário não configura cada efeito individualmente. O controle disponível é a chave **Usar efeitos esperados**, que liga ou desliga a validação automática daquele passo.

Mantenha **Usar efeitos esperados** ligado quando a ação precisa provar que a tela mudou. Exemplos:

- clique que abre menu;
- botão que abre modal;
- ação que navega para outra página;
- ação que abre nova aba;
- hover que revela submenu;
- clique que precisa deixar o próximo campo visível.

Desligue **Usar efeitos esperados** naquele passo quando:

- a página é muito dinâmica e o efeito aponta para um item volátil;
- o efeito depende de dados que mudam sempre;
- a tela abre o mesmo container, mas o próximo alvo real depende de outro passo;
- o passo já possui uma espera explícita mais adequada.

Quando precisar de controle manual, use ações como `wait` e `assert`. Elas são mais indicadas para esperas e validações definidas diretamente pelo usuário.

### Promoção Gradual

Efeitos esperados gravados pelo Smart Web Flow começam de forma conservadora. Quando o QANode observa o mesmo cenário rodando com sucesso e o mesmo alvo sendo confirmado repetidas vezes, ele pode transformar essa pista em uma validação mais forte.

Isso evita que um efeito criado por timing, mudança momentânea de DOM ou item dinâmico quebre cenários logo na primeira execução.

---

## Self Healing

Quando **Self Healing** está habilitado, o Smart Web Flow tenta recuperar o alvo caso as estratégias gravadas não encontrem mais o elemento.

Ele pode ajudar quando:

- o texto mudou levemente;
- um botão mudou de posição;
- um ID deixou de existir, mas o elemento continua com o mesmo papel;
- um link mudou o texto, mas manteve contexto e destino semelhantes;
- um campo foi renderizado por outro componente visual.

Ele evita aplicar recuperação quando:

- existem muitos candidatos parecidos;
- a ação pode ser destrutiva e o alvo não está claro;
- o contexto mudou de forma incompatível;
- o melhor candidato não é claramente melhor que o segundo.

Quando uma recuperação é usada, a execução mostra o item em **Self-healing aplicado**. O usuário pode revisar e aplicar ao fluxo quando fizer sentido.

> **Importante:** Self Healing não substitui uma boa gravação. Ele é uma rede de segurança para mudanças comuns de UI, não uma garantia de que qualquer tela alterada será interpretada corretamente.

---

## Otimizações Após Execução

Depois de execuções bem-sucedidas, o QANode pode sugerir melhorias para o Smart Web Flow.

As sugestões mais comuns são:

- promover uma estratégia que funcionou melhor;
- adicionar espera antes de um alvo que apareceu tarde;
- registrar um frame/iframe detectado automaticamente;
- remover hover opcional que não foi necessário para a execução passar;
- ajustar validações de efeito esperado que foram confirmadas.

Essas otimizações aparecem nos detalhes da execução. Revise antes de aplicar.

O QANode evita mostrar a mesma sugestão repetidamente quando a definição do fluxo não mudou. Se você alterar passos, seletores ou dados relevantes da UI, novas sugestões podem aparecer.

---

## Ações Disponíveis

| Ação | Cor | Descrição |
|------|-----|-----------|
| [navigate](#navigate) | 🔵 Azul | Navega para uma URL |
| [switchPage](#switchpage) | 🟣 Índigo | Alterna para outra página/aba |
| [frame](#frame) | Cinza | Alterna para um iframe ou volta para a página principal |
| [click](#click-e-dblclick) | 🟡 | Clica em um elemento |
| [dblclick](#click-e-dblclick) | 🟡 | Executa duplo clique |
| [fill](#fill) | 🟢 | Preenche um campo |
| [check](#check-e-uncheck) | 🟢 Esmeralda | Marca checkbox/radio |
| [uncheck](#check-e-uncheck) | 🟢 Esmeralda | Desmarca checkbox |
| [selectOption](#selectoption) | 🟣 | Seleciona opção em dropdown |
| [hover](#hover) | 🔵 Ciano | Passa o mouse sobre um elemento |
| [focus](#focus) | 🔵 Azul claro | Coloca foco em um elemento |
| [press](#press) | 🟢 | Pressiona uma tecla |
| [setSlider](#setslider) | 🟣 | Ajusta slider/range |
| [drag](#drag-and-drop) | Rosa | Arrasta e solta um elemento |
| [scroll](#scroll) | 🟡 | Rola a página ou um container |
| [refresh](#refresh) | 🟠 | Recarrega a página |
| [clock](#clock) | 🟢 Água | Controla o relógio JavaScript do navegador |
| [wait](#wait) | 🟣 | Aguarda tempo ou condição |
| [assert](#assert) | 🔴 | Verifica uma condição |
| [extract](#extract) | 🔵 Ciano | Extrai um valor |
| [extractList](#extractlist) | 🟢 Esmeralda | Extrai itens repetidos |
| [extractTable](#extracttable) | 🟤 | Extrai tabela/grid |
| [uploadFile](#uploadfile) | 🟤 | Envia arquivo para um input de upload |

---

## Detalhes das Ações

### navigate

Navega para uma URL.

| Campo | Descrição |
|-------|-----------|
| **URL** | URL de destino. Aceita expressões `{{ }}` |

Use `navigate` no início do fluxo ou quando desejar ir diretamente para uma rota específica.

---

### switchPage

Alterna a execução para outra aba/janela aberta pelo navegador.

Use quando um clique abre uma nova aba e os próximos passos devem continuar nela.

| Campo | Descrição |
|-------|-----------|
| **Modo** | Define como escolher a página: última aberta, por URL ou por título |
| **URL / Título** | Critério usado quando o modo exige correspondência |
| **Wait Until** | Estado de carregamento esperado |
| **Timeout** | Tempo máximo para encontrar a página |

---

### frame

Alterna o contexto para um iframe ou volta para a página principal.

Use quando a aplicação renderiza campos ou botões dentro de iframes.

| Campo | Descrição |
|-------|-----------|
| **Modo** | `main`, URL, nome ou seletor do iframe |
| **Frame URL / Nome** | Identificador do iframe |
| **Timeout** | Tempo máximo para localizar o frame |

> O Smart Web pode detectar iframe automaticamente em alguns casos e sugerir a inclusão de um passo `frame` para tornar futuras execuções mais rápidas.

---

### click e dblclick

Executam clique ou duplo clique em um elemento.

| Campo | Descrição |
|-------|-----------|
| **Locator** | Localizador semântico principal |
| **Referência do alvo** | Restringe a busca a uma linha, card, item ou registro específico |
| **Estratégias CSS/XPath** | Seletores alternativos gravados |
| **Tentativas** | Número de ciclos de tentativa do passo |
| **Usar efeitos esperados** | Liga/desliga a validação automática do resultado observado no clique |

Mantenha **Usar efeitos esperados** ligado para cliques que abrem menu, modal, página, dropdown ou próximo campo.

---

### fill

Preenche um campo de texto, input, textarea ou componente editável.

| Campo | Descrição |
|-------|-----------|
| **Locator** | Campo a preencher |
| **Texto** | Valor literal ou expressão `{{ }}` |
| **Limpar primeiro** | Limpa o campo antes de preencher |

Para editores ricos, como campos com `contenteditable`, prefira gravar pelo Smart Web Flow e revise se o alvo gravado representa a área real de edição.

---

### check e uncheck

Marca ou desmarca checkboxes, radios e controles equivalentes.

O Smart Web Flow tenta lidar com controles estilizados, quando o input real está oculto mas existe um label ou container clicável.

---

### selectOption

Seleciona uma opção em um `<select>` ou dropdown compatível.

| Campo | Descrição |
|-------|-----------|
| **Selecionar por** | `value`, `label` ou `index` |
| **Valor / Label / Índice** | Opção desejada |

Para dropdowns customizados que não são `<select>`, a extensão normalmente grava uma sequência de `click`/`press`/`wait` em vez de `selectOption`.

No modo Inspect da extensão, selects nativos mostram as opções disponíveis em uma lista clicável. Assim o usuário escolhe a opção visualmente em vez de digitar o valor manualmente.

---

### hover

Passa o mouse sobre um elemento.

Use quando um menu ou submenu aparece apenas ao passar o mouse.

O Smart Web Flow trata hover como uma ação curta e objetiva. Se o próximo alvo já estiver visível sem hover, a execução pode sugerir remover o hover opcional para reduzir ruído no fluxo.

---

### focus

Coloca foco em um elemento.

É útil antes de `press`, em campos que reagem ao foco, ou em componentes customizados que só aceitam teclado depois de focados.

---

### press

Pressiona uma tecla ou combinação.

Exemplos:

```
Enter
Escape
Tab
Control+A
Meta+S
ArrowDown
```

Use `press` para confirmar formulários, navegar em menus, selecionar opções por teclado ou acionar atalhos da aplicação.

---

### setSlider

Ajusta um slider/range.

| Campo | Descrição |
|-------|-----------|
| **Valor** | Valor final desejado quando o componente suporta valor |
| **Percentual** | Posição proporcional do slider |
| **Passos** | Quantidade de movimentos usados durante o ajuste |

---

### drag and drop

Arrasta um elemento de origem para um destino.

O Smart Web Flow grava e executa drag/drop usando uma combinação de:

- localizador da origem;
- localizador ou contexto do destino;
- identidade do card/item;
- coordenadas como fallback;
- ajustes de movimento para componentes customizados.

### Dicas para drag/drop

- Prefira gravar a ação real pela extensão.
- Quando o destino é uma coluna, lista ou lane, revise se o alvo gravado representa o container e não apenas outro card.
- Para cards dinâmicos, use **Referência do alvo** ou variáveis para identificar o card correto.
- Se a aplicação precisa de tempo para estabilizar depois do drop, adicione `waitAfter` ou uma espera explícita.
- Se a extensão avisar que não conseguiu mover o item durante a gravação, ainda assim revise o passo no QANode: em sites muito customizados, a execução pode conseguir reproduzir o drag/drop mesmo quando a extensão não consegue mover visualmente o card.

---

### scroll

Rola a página ou um container.

| Modo | Descrição |
|------|-----------|
| **To selector** | Rola até um elemento específico |
| **To position** | Rola para coordenadas X/Y |
| **By pixels** | Rola uma quantidade de pixels |
| **To bottom** | Rola até o fim |

---

### refresh

Recarrega a página atual.

| Campo | Descrição |
|-------|-----------|
| **Wait Until** | Estado de carregamento esperado |
| **Wait After** | Espera adicional após o reload |

---

### clock

Controla o relógio JavaScript do navegador usando a API de clock do Playwright.

| Operação | Descrição |
|----------|-----------|
| **setFixedTime** | Fixa a data/hora do navegador |
| **pauseAt** | Avança até uma data/hora e pausa |
| **fastForward** | Avança o tempo por uma duração |
| **resume** | Retoma o relógio normal |

Use em cenários que dependem de datas, timers, expiração, contagem regressiva ou regras por horário.

> O `clock` controla o tempo JavaScript observado pela página. Ele não altera o horário do servidor nem respostas de APIs externas.

---

### wait

Aguarda uma condição antes de continuar.

Modos comuns:

- tempo fixo;
- elemento visível;
- elemento oculto;
- elemento habilitado;
- texto presente;
- URL ou título esperado.

Use waits para estabilizar páginas lentas, pós-navegação, grids que carregam por API ou componentes que aparecem depois de animações.

---

### assert

Valida uma condição e registra o resultado.

Modos disponíveis incluem:

- visível / oculto;
- texto contém / texto igual;
- quantidade esperada;
- habilitado / desabilitado;
- marcado;
- valor esperado;
- atributo esperado;
- URL;
- título.

Asserts são importantes para transformar uma automação em teste: eles provam que o resultado esperado apareceu.

Quando a validação está dentro de uma tabela, lista ou card repetido, combine o assert com **Referência do alvo** para validar o registro certo, e não o primeiro texto parecido da tela.

---

### extract

Extrai um valor único da página.

| Campo | Descrição |
|-------|-----------|
| **Nome** | Nome do output |
| **Atributo** | `text`, `inputValue`, `href`, `src` ou atributo específico |

Quando o passo vem da extensão, o Smart Web Flow trata extração como valor dinâmico. O texto visto durante a gravação é usado como amostra/evidência, mas o alvo deve ser encontrado por um seletor estável, por contexto ou pela posição correta entre elementos equivalentes.

Isso ajuda em telas onde o valor pode mudar a cada execução, por exemplo contadores, status, saldos, totais e indicadores.

Quando a extração está dentro de uma linha, card ou item repetido, use **Referência do alvo** para extrair apenas o valor do registro correto.

O valor extraído fica disponível em:

```
{{ steps["Nome do Nó"].outputs.extracts.nome }}
```

---

### extractList

Extrai dados de elementos repetidos, como cards, linhas, resultados ou itens de uma lista.

Configure:

- seletor do item repetido;
- campos dentro de cada item;
- nome e atributo de cada campo;
- limite máximo de itens.

---

### extractTable

Extrai dados tabulares de uma tabela ou grid.

Use quando a tela possui uma tabela HTML, grid estruturado ou lista tabular que deve virar saída estruturada.

O resultado pode ser usado em nós posteriores, asserts, relatórios ou expressões.

---

### uploadFile

Envia um arquivo para um campo `<input type="file">`.

| Campo | Descrição |
|-------|-----------|
| **Arquivo** | `fileRef` a ser enviado |
| **Locator / Estratégias** | Alvo do input de arquivo |
| **Timeout** | Tempo máximo para localizar o input |

O passo é gravado automaticamente pela extensão quando o usuário escolhe um arquivo em um input de upload. Ao importar o fluxo no QANode, se o arquivo original ainda não estiver anexado ao fluxo, o passo fica destacado para o usuário escolher o `fileRef` antes de executar.

Use `uploadFile` para cenários como:

- enviar documento em cadastro;
- anexar comprovante;
- subir imagem, PDF, CSV ou planilha;
- testar endpoints web com upload por formulário.

---

## Downloads

O Smart Web Flow não possui uma ação separada de download. Quando um passo de **click** dispara download no navegador, o QANode captura o arquivo automaticamente e o expõe como `fileRef`.

Arquivos baixados ficam disponíveis no output `downloads` e, quando houver apenas um download relevante, também podem aparecer como `fileRef` no painel de variáveis.

Exemplo:

```
{{ steps["smart-web-flow"].outputs.downloads.relatorio }}
{{ steps["smart-web-flow"].outputs.fileRef }}
```

Use o `fileRef` baixado em outros nós, como **Extrair arquivo**, **HTTP Request**, **SSH Command**, **Mobile Flow** ou **Custom JavaScript**.

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `sessionId` | `string` | ID da sessão web aberta ou reutilizada |
| `extracts` | `object` | Valores extraídos por passos `extract`, `extractList` e `extractTable` |
| `asserts` | `object` | Resultado das validações |
| `downloads` | `object` | Arquivos baixados durante cliques, como `fileRef` |
| `fileRef` | `fileRef` | Atalho para arquivo baixado quando aplicável |

---

## Evidências por Passo

Cada passo pode capturar screenshot.

| Campo | Descrição |
|-------|-----------|
| **Tirar screenshot** | Ativa/desativa evidência do passo |
| **Modo da captura** | Antes, depois ou ambos |
| **Aguardar antes da captura** | Espera por carregamento antes do screenshot |
| **Delay** | Tempo extra antes da captura |

Use screenshots em ações críticas, validações, transições de tela e pontos que precisam aparecer no relatório.

---

## Auditoria de Performance

Quando habilitada, a auditoria de performance coleta informações por tela:

- tempo de carregamento;
- FCP e LCP;
- quantidade de requests;
- APIs lentas;
- erros de rede/API;
- gráficos e relatório de performance.

O Smart Web Flow cria checkpoints por navegação, refresh, troca de aba e ações que recarregam a página.

---

## Acessibilidade

Quando a varredura de acessibilidade está habilitada, o QANode executa uma análise da página e gera evidências com os problemas encontrados.

Use esse recurso para complementar testes funcionais com checks básicos de acessibilidade, especialmente em telas críticas.

---

## Gravando com a Extensão Chrome

O jeito mais simples de criar um Smart Web Flow é usar o **QANode Recorder** no Chrome.

1. Instale a extensão: [QANode Recorder na Chrome Web Store](https://chromewebstore.google.com/detail/qanode-recorder/gmdklmpafacfkjmljgnoafcjjcdjiajo)
2. Abra a página que deseja testar.
3. Selecione o modo **Smart Web Flow** no popup da extensão.
4. Escolha o modo de captura:
   - **Touch mode** para gravar interagindo normalmente com a página;
   - **Inspect mode** para selecionar elementos, revisar ações sugeridas e gravar passos com mais controle.
5. Clique em **Start**.
6. Execute o fluxo manualmente no site ou use os painéis do Inspect mode para montar as ações.
7. Use os atalhos de gravação quando precisar:
   - `Ctrl+A` para assert;
   - `Ctrl+E` para extract;
   - `Ctrl+Shift+E` para extract list;
   - `Ctrl+Alt+W` para wait;
   - `Ctrl+Alt+T` para extract table.
8. Clique em **Stop** ou finalize pelo painel do Inspect mode.
9. Copie o JSON.
10. Cole no canvas do QANode com `Ctrl+V`.

A extensão grava:

- passos executados;
- screenshots, quando configurado;
- localizadores e seletores alternativos;
- contexto de escopo;
- identidade do alvo;
- iframes;
- sessão/storage, quando habilitado;
- efeitos esperados para menus, overlays, navegação e próximos alvos.
- passos de upload de arquivo, quando a página usa input de arquivo.

---

## Dicas de Estabilidade

### Prefira intenção a posição

Evite depender apenas de `nth` ou de coordenadas. Quando houver elementos repetidos, use texto de escopo, variáveis, labels ou atributos estáveis.

### Use waits nos lugares certos

Se uma tela demora para carregar, adicione `wait` antes do alvo que aparece tarde. Evite aumentar timeouts em todos os passos sem necessidade.

### Revise efeitos esperados

Efeitos esperados são úteis, mas devem representar o resultado real da ação. Se a validação automática daquele passo ficar instável, desligue **Usar efeitos esperados** e use um `wait` ou `assert` explícito para validar o ponto correto.

### Dê bons nomes aos passos

Labels claras ajudam a ler logs, entender falhas e usar expressões depois.

### Use variáveis para dados dinâmicos

Se o card, chamado, cliente ou registro muda por execução, substitua o valor gravado por `{{ variables... }}` ou output de passo anterior.

### Melhore a aplicação quando possível

Quando você controla a aplicação testada, adicionar `data-testid`, `aria-label`, labels reais e roles corretos melhora muito a estabilidade da automação.

---

## Limitações

- O Smart Web Flow automatiza o navegador; ele não valida regras de servidor sozinho.
- O `clock` altera o relógio JavaScript da página, não o horário de APIs externas.
- Sites com CSP muito restritivo, iframes cross-origin ou componentes fechados podem limitar a gravação.
- Interfaces extremamente ambíguas podem exigir ajuste manual de escopo, seletor ou espera.
- Self Healing não deve ser usado como substituto para seletores, labels e dados de teste bem definidos.

---

## Próximos Passos

- [Extensão Chrome](../../desktop/extensao-chrome.md) — Como gravar cenários web
- [Execução e Depuração](../../editor-fluxos/execucao-depuracao.md) — Como analisar falhas e evidências
- [Expressões](../../expressoes/expressoes.md) — Como usar variáveis e outputs nos passos
