# MCP — Integração com IA

> **Disponível em:** QANode Desktop Community e Enterprise. As ferramentas acessíveis dependem da edição e das permissões do usuário.

O **MCP** permite conectar ferramentas de IA ao QANode para que o usuário trabalhe com projetos, cenários, execuções, dashboards e defeitos diretamente a partir da IDE, chat corporativo ou outro cliente compatível.

Na prática, o cliente conectado ao MCP chama recursos do QANode usando as permissões do usuário autenticado. Tudo que é criado ou alterado continua existindo dentro do QANode como objeto normal da plataforma.

---

## Quando Usar

Use o MCP quando você quer trabalhar com o QANode sem sair do ambiente onde já está investigando, documentando ou implementando:

- criar ou ajustar cenários no QANode com assistência de IA;
- automatizar fluxos que passam por web, API, banco, arquivos, SSH ou mobile;
- investigar um cenário existente que está falhando;
- transformar documentação de projeto em cenários iniciais;
- criar dashboards ou consultar métricas do QANode por linguagem natural;
- abrir ou analisar defeitos enquanto trabalha no código.

O MCP não substitui o editor visual. Tudo que é criado continua aparecendo no QANode como projeto, cenário, nó, execução, dashboard, suíte ou defeito normal.

---

## Como Conectar

No **Desktop Community**, abra **Conectar IA** no aplicativo. Essa tela orienta a conexão com clientes compatíveis e mantém o MCP local disponível enquanto o QANode estiver aberto.

Em uma instalação **Enterprise**, use um token de acesso e a URL da API:

1. Gere um token em **Configurações → Tokens de Acesso**
2. Configure o cliente MCP compatível
3. Informe a URL da API do QANode
4. Envie o token no header `Authorization`

Endpoint:

```text
https://qanode.empresa.com/api/mcp
```

Header:

```text
Authorization: Bearer qnt_xxxxx
```

Exemplo genérico:

```json
{
  "mcpServers": {
    "qanode": {
      "url": "https://qanode.empresa.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ${QANODE_TOKEN}"
      }
    }
  }
}
```

O formato exato depende do cliente compatível, mas a URL e o header são os mesmos.

---

## Segurança e Permissões

O MCP respeita as mesmas permissões do QANode.

| Área | Regra |
|------|-------|
| Projetos | Acesso limitado aos projetos permitidos ao usuário |
| Cenários | Criação, edição e execução seguem as permissões do usuário |
| Execuções | A consulta de runs respeita permissões de visualização |
| Credenciais | Segredos nunca são retornados em texto claro |
| Variáveis secretas | Valores secretos são mascarados |
| Dashboards | Consultas e widgets respeitam as permissões de dados |
| Defeitos | Workflow, campos obrigatórios e transições continuam valendo |

O token não concede acesso extra. Ele apenas autentica o cliente conectado como aquele usuário ou integração.

---

## O Que o MCP Permite Fazer

| Área | O que fica disponível |
|------|-----------------|
| **Projetos** | Buscar projetos, consultar documentos anexados e usar esse contexto em cenários ou diagnósticos |
| **Cenários** | Criar fluxos, importar nós, adicionar etapas, conectar, validar e executar |
| **Smart Web** | Criar automações web, validar telas, extrair dados, usar tools WebMCP e preparar o estado inicial do navegador quando solicitado |
| **Mobile** | Criar jornadas Android/iOS com Appium e salvar como Mobile Flow |
| **Banco de Dados** | Consultar schema e criar validações ou consultas em nós de banco |
| **Arquivos** | Gerar, extrair e reaproveitar arquivos em fluxos |
| **Variáveis** | Criar variáveis globais ou de projeto para reutilização |
| **Credenciais** | Cadastrar credenciais a partir dos dados fornecidos pelo usuário |
| **Suítes** | Criar, editar, executar, agendar e diagnosticar falhas |
| **Defeitos** | Abrir, comentar, anexar, tramitar e investigar defeitos |
| **Dashboards** | Criar painéis, widgets e consultas rápidas sobre dados do QANode |
| **Componentes** | Criar componentes reutilizáveis para fluxos recorrentes |

---

## Como Uma Solicitação Funciona

Quando o usuário solicita uma ação pelo cliente conectado, o fluxo geral é:

1. O cliente envia a solicitação ao QANode pelo MCP.
2. O QANode valida autenticação, permissões e dados necessários.
3. A ação é aplicada nos objetos reais do QANode, como cenário, execução, dashboard ou defeito.
4. O resultado volta para o cliente com os dados disponíveis para aquele usuário.

Para cenários e testes, informações como projeto, sistema alvo, credencial, dados de teste, resultado esperado e regras de limpeza ajudam o QANode a criar um fluxo mais próximo do uso real.

Exemplo:

```text
No projeto Task Manager, crie um cenário que acesse localhost:3008,
faça login com o usuário de teste, crie uma tarefa, marque como concluída
e valide no banco que o status ficou completed.
```

Ações sensíveis, como usar credenciais, criar massa em sistemas reais, executar cenários destrutivos ou tramitar defeitos, devem ser tratadas com o mesmo cuidado que teriam dentro do QANode.

---

## Smart Web e Referência do Alvo

Em páginas com tabelas, cards, kanban ou listas repetidas, é comum existir o mesmo botão ou status várias vezes na tela.

A **Referência do alvo** do Smart Web registra qual linha, card ou item deve servir de contexto antes de clicar, validar ou extrair um valor. Isso vale tanto para fluxos criados pelo editor quanto para fluxos criados ou ajustados por MCP.

Exemplo:

```text
Valide que o status está Pago no pedido criado nesta execução,
não em qualquer pedido antigo da tela.
```

Isso ajuda o QANode a agir no registro correto mesmo quando os dados mudam a cada execução.

---

## Sessões Web e Mobile

Quando um cenário é dividido em mais de um nó Smart Web ou Mobile, a reutilização de sessão define se os nós continuam no mesmo navegador, contexto web ou sessão mobile.

Isso é útil quando:

- o primeiro nó faz login e o segundo continua autenticado;
- uma compra ou cadastro começa em uma tela e termina em outra;
- o app mobile precisa manter o mesmo estado entre partes do fluxo.

Para sessões web já autenticadas, como sistemas enterprise com SSO, sessões persistidas com nome permitem reaproveitar cookies e localStorage. Para fluxos comuns que fazem login durante a própria execução, uma sessão temporária por run costuma ser suficiente.

Quando um nó posterior precisa reutilizar a sessão aberta por um nó anterior, o **Armazenamento da sessão** deve manter essa sessão disponível. O comportamento é o mesmo no editor e em ações feitas via MCP.

---

## Nós Ignorados em Execuções

Nós marcados como **Ignorar na execução** continuam visíveis no canvas, mas execuções normais registram esses nós como pulados e seguem o fluxo sem executá-los.

Esse comportamento também vale para execuções iniciadas ou cenários editados via MCP. Ele é útil para rascunhos, experimentos e partes temporariamente isoladas do fluxo, mas não substitui a correção de uma falha real.

---

## Documentos de Projeto

O MCP expõe documentos anexados ao projeto como contexto para criar cenários, tirar dúvidas ou entender regras de negócio.

Para documentos grandes, o QANode permite pesquisar primeiro por trechos relevantes dentro dos anexos do projeto. Assim, o cliente encontra páginas, seções ou partes do texto antes de abrir o documento completo.

O QANode consegue ler:

- PDF com texto;
- PDF escaneado ou imagem com OCR;
- DOCX;
- TXT, JSON e CSV;
- XLSX com prévia de planilhas;
- metadados de outros arquivos.

Quando o trecho encontrado não for suficiente, o cliente pode solicitar uma página, um intervalo de páginas ou o documento completo.

Quando a busca está limitada a um projeto, ela também pode considerar nomes, descrições e anotações dos cenários permitidos ao usuário. Assim, decisões e regras registradas no próprio canvas podem ser encontradas junto com os documentos anexados.

---

## Documentação de Cenários Criados por IA

Ao criar um cenário pelo MCP, o QANode adiciona uma anotação padronizada ao canvas com:

- nome do cenário;
- descrição;
- pré-condições;
- casos de teste;
- resultado esperado.

Cada nó do cenário representa um caso de teste. O nome do nó descreve a ação, como **Preencher formulário no site** ou **Consultar status no banco**, enquanto a anotação mantém a numeração `CT-01`, `CT-02` e assim por diante.

Se o usuário informar um nome exato para o cenário ou para um nó, o cliente deve preservá-lo. Quando nenhum nome é fornecido, o cenário recebe um título curto que descreve seu propósito, como **Validação de login**, e os nós recebem rótulos orientados à ação.

A lista de casos de teste permanece sincronizada quando os nós são adicionados, renomeados, importados ou removidos pelo MCP. Anotações comuns já existentes no canvas não são substituídas; a documentação padrão é mantida como uma anotação própria do cenário.

---

## Documentação Oficial do QANode

Além dos documentos do seu projeto, o MCP também pode consultar a documentação oficial do QANode.

Esse recurso ajuda a entender como configurar um nó, confirmar um comportamento do produto ou localizar a página certa da documentação sem sair do cliente conectado.

Exemplos:

```text
Como eu configuro um Smart Web Flow com sessão persistida?
```

```text
Quais opções existem para criar widgets de dashboard?
```

A busca usa o idioma solicitado quando disponível e retorna os trechos mais relevantes da documentação.

---

## Dashboards

Pelo MCP, o usuário pode criar dashboards ou consultar métricas do QANode por linguagem natural.

Exemplos:

```text
Quantas execuções tivemos em maio?
```

```text
Crie um dashboard público com total de execuções no mês,
taxa de sucesso, falhas recentes e evolução diária.
```

Para widgets simples, o QANode usa o construtor visual. Para consultas mais complexas, o modo SQL pode ser usado quando o usuário tem permissão.

Em clientes compatíveis com **MCP Apps**, consultas numéricas também podem abrir uma visualização interativa com tabela, métrica ou gráficos compatíveis com o resultado. O QANode oferece apenas formatos adequados às colunas retornadas e mantém a resposta em texto ou dados estruturados quando o cliente não suporta essa interface.

---

## Acompanhamento Visual de Execuções

Clientes compatíveis com **MCP Apps** podem mostrar um inspetor visual quando uma execução de cenário ou suíte é iniciada ou consultada pelo MCP.

O inspetor reúne:

- status e duração;
- progresso dos cenários e nós;
- rótulos definidos no canvas;
- logs principais em seções recolhíveis;
- falha e mensagem do ponto afetado;
- evidências da execução atual;
- indicação dos maiores tempos da execução.

Em suítes, os nós são agrupados por cenário para preservar o contexto, inclusive em execuções sequenciais ou paralelas. Enquanto a execução estiver em andamento, o cliente pode atualizar a visualização até o resultado final. Se o cliente não oferecer suporte a MCP Apps, os mesmos dados continuam disponíveis em formato textual ou estruturado.

---

## WebMCP e Estado Inicial do Navegador

Ao gravar um Smart Web Flow em uma página Chromium compatível, o cliente pode listar as tools WebMCP declaradas pelo site, chamar uma tool com os parâmetros definidos por ela e validar se a capacidade está disponível naquele estado da página. A chamada é executada durante a gravação, e seu resultado estruturado fica disponível no output `webmcpResult` após a execução do cenário.

O MCP também consegue configurar cookies, `localStorage`, `sessionStorage`, headers da sessão e JavaScript de inicialização antes da primeira navegação. Essa configuração é opcional e só deve ser usada quando o usuário pedir explicitamente um estado inicial; ela não substitui passos normais de interação com a interface.

Use variáveis protegidas para segredos ou valores específicos do ambiente. Para detalhes dos campos e dos passos, consulte [Smart Web Flow](../nos/web/smart-web-flow.md).

---

## Defeitos

Com MCP, o usuário pode investigar e acompanhar defeitos a partir do cliente conectado.

Exemplos:

- abrir defeito a partir de uma execução com falha;
- analisar logs e evidências da execução original;
- baixar anexos do defeito;
- comentar quando necessário;
- tramitar para uma fila ou usuário, quando permitido;
- abrir sandbox para investigar sem afetar o cenário oficial.

O diagnóstico volta para o cliente conectado. Ele não vira comentário público no defeito automaticamente.

---

## Boas Práticas

- Use tokens diferentes para CI/CD e MCP quando quiser separar auditoria.
- Revogue tokens que não estão mais em uso.
- Prefira credenciais salvas em vez de colar senhas no prompt.
- Defina validações explícitas quando o objetivo for um cenário de teste.
- Revise cenários criados por automação antes de usar em suítes críticas.
- Use dashboards por papel quando o painel tiver dados de uma equipe específica.
- Em defeitos, revise o diagnóstico antes de tramitar ou comentar.

---

## Próximos Passos

- [Tokens de Integração](../ci-cd/tokens-integracao.md) — Gere e governe tokens usados por CI/CD e MCP
- [Smart Web Flow](../nos/web/smart-web-flow.md) — Entenda gravação, localizadores e referência do alvo
- [Mobile Flow](../nos/mobile/mobile-flow.md) — Entenda gravação mobile com Appium
- [Dashboard - Enterprise](../dashboard/dashboard-enterprise.md) — Veja painéis, widgets e SQL
- [Defeitos - Visão Geral](../defeitos/visao-geral.md) — Veja workflow, sandbox e anexos
