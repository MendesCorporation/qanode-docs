# MCP — Integração com IA

> **Disponível em:** QANode Enterprise

O **MCP** permite conectar ferramentas de IA ao QANode para que o usuário possa pedir ações diretamente a partir da IDE, chat corporativo ou outro cliente compatível.

Com essa integração, a IA pode operar o QANode usando as permissões do usuário autenticado: criar cenários, ajustar fluxos, consultar execuções, ler documentação de projetos, criar dashboards, investigar falhas, trabalhar com defeitos e usar os nós visuais da plataforma.

---

## Quando Usar

Use o MCP quando você quer:

- pedir para uma IA criar ou ajustar cenários no QANode;
- automatizar fluxos que passam por web, API, banco, arquivos, SSH ou mobile;
- investigar um cenário existente que está falhando;
- transformar documentação de projeto em cenários iniciais;
- criar dashboards ou consultar métricas do QANode por linguagem natural;
- abrir ou analisar defeitos enquanto trabalha no código.

O MCP não substitui o editor visual. Tudo que é criado continua aparecendo no QANode como projeto, cenário, nó, execução, dashboard, suíte ou defeito normal.

---

## Como Conectar

1. Gere um token em **Configurações → Tokens de Acesso**
2. Configure o cliente MCP usado pela sua IA
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

O formato exato depende do cliente de IA, mas a URL e o header são os mesmos.

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

O token não concede acesso extra. Ele apenas autentica a IA como aquele usuário ou integração.

---

## O Que Você Pode Pedir

| Área | Exemplos de uso |
|------|-----------------|
| **Projetos** | Buscar projetos, pesquisar trechos em documentos e ler anexos quando necessário |
| **Cenários** | Criar fluxos, importar nós, adicionar etapas, conectar, validar e executar |
| **Smart Web** | Gravar navegação web, validar telas, extrair dados e reaproveitar sessão quando necessário |
| **Mobile** | Gravar jornadas Android/iOS com Appium e salvar como Mobile Flow |
| **Banco de Dados** | Consultar schema e criar validações ou consultas em nós de banco |
| **Arquivos** | Gerar, extrair e reaproveitar arquivos em fluxos |
| **Variáveis** | Criar variáveis globais ou de projeto para reutilização |
| **Credenciais** | Criar credenciais quando o usuário fornece os dados necessários |
| **Suítes** | Criar, editar, executar, agendar e diagnosticar falhas |
| **Defeitos** | Abrir, comentar, anexar, tramitar e investigar defeitos |
| **Dashboards** | Criar painéis, widgets e consultas rápidas sobre dados do QANode |
| **Componentes** | Criar componentes reutilizáveis quando o usuário pedir explicitamente |

---

## Como Pedir

Quanto mais claro for o pedido, melhor o resultado ao usar o QANode por um cliente de IA. Inclua:

- projeto onde o cenário deve ser criado;
- URL ou sistema alvo;
- credenciais ou nome da credencial já salva;
- dados de teste permitidos;
- resultado esperado;
- regras de limpeza, como excluir massa criada ao final;
- se o fluxo deve ser apenas criado ou também executado.

Exemplo:

```text
No projeto Task Manager, crie um cenário que acesse localhost:3008,
faça login com o usuário de teste, crie uma tarefa, marque como concluída
e valide no banco que o status ficou completed.
```

Quando faltar informação importante, confirme os dados antes de criar registros, mexer em dados reais ou executar ações destrutivas.

---

## Smart Web e Referência do Alvo

Em páginas com tabelas, cards, kanban ou listas repetidas, é comum existir o mesmo botão ou status várias vezes na tela.

Nesses casos, use a **Referência do alvo** do Smart Web. Ela indica qual linha, card ou item deve ser usado como referência antes de clicar, validar ou extrair um valor.

Exemplo:

```text
Valide que o status está Pago no pedido criado nesta execução,
não em qualquer pedido antigo da tela.
```

Isso ajuda o QANode a agir no registro correto mesmo quando os dados mudam a cada execução.

---

## Sessões Web e Mobile

Quando um cenário é dividido em mais de um nó Smart Web ou Mobile, informe se os nós precisam continuar na mesma sessão.

Use isso quando:

- o primeiro nó faz login e o segundo continua autenticado;
- uma compra ou cadastro começa em uma tela e termina em outra;
- o app mobile precisa manter o mesmo estado entre partes do fluxo.

Para sessões web já autenticadas, como sistemas enterprise com SSO, use sessões persistidas com nome. Para fluxos comuns que fazem login durante a própria execução, uma sessão temporária por run costuma ser suficiente.

---

## Documentos de Projeto

O MCP permite que clientes de IA usem documentos anexados ao projeto como contexto para criar cenários, tirar dúvidas ou entender regras de negócio.

Para documentos grandes, a IA pode pesquisar primeiro por trechos relevantes dentro dos anexos do projeto. Assim, ela encontra páginas, seções ou partes do texto antes de abrir o documento completo.

O QANode consegue ler:

- PDF com texto;
- PDF escaneado ou imagem com OCR;
- DOCX;
- TXT, JSON e CSV;
- XLSX com prévia de planilhas;
- metadados de outros arquivos.

Quando o trecho encontrado não for suficiente, peça para a IA ler uma página, um intervalo de páginas ou o documento completo.

---

## Documentação Oficial do QANode

Além dos documentos do seu projeto, o MCP também pode consultar a documentação oficial do QANode.

Use isso quando quiser perguntar como configurar um nó, entender um recurso ou confirmar o comportamento de uma área do produto sem sair do cliente de IA.

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

Você pode pedir para a IA criar dashboards ou responder perguntas pontuais sobre os dados do QANode.

Exemplos:

```text
Quantas execuções tivemos em maio?
```

```text
Crie um dashboard público com total de execuções no mês,
taxa de sucesso, falhas recentes e evolução diária.
```

Para widgets simples, o QANode usa o construtor visual. Para consultas mais complexas, o modo SQL pode ser usado quando o usuário tem permissão.

---

## Defeitos

Com MCP, um cliente de IA pode ajudar a investigar e acompanhar defeitos.

Exemplos:

- abrir defeito a partir de uma execução com falha;
- analisar logs e evidências da execução original;
- baixar anexos do defeito;
- comentar quando o usuário pedir;
- tramitar para uma fila ou usuário, quando permitido;
- abrir sandbox para investigar sem afetar o cenário oficial.

O diagnóstico volta no chat. Ele não vira comentário público no defeito automaticamente.

---

## Boas Práticas

- Use tokens diferentes para CI/CD e MCP quando quiser separar auditoria.
- Revogue tokens que não estão mais em uso.
- Prefira credenciais salvas em vez de colar senhas no prompt.
- Peça validações explícitas quando o objetivo for um cenário de teste.
- Revise cenários criados por automação antes de usar em suítes críticas.
- Use dashboards por papel quando o painel tiver dados de uma equipe específica.
- Em defeitos, peça para a IA explicar o diagnóstico antes de tramitar ou comentar.

---

## Próximos Passos

- [Tokens de Integração](../ci-cd/tokens-integracao.md) — Gere e governe tokens usados por CI/CD e MCP
- [Smart Web Flow](../nos/web/smart-web-flow.md) — Entenda gravação, localizadores e referência do alvo
- [Mobile Flow](../nos/mobile/mobile-flow.md) — Entenda gravação mobile com Appium
- [Dashboard - Enterprise](../dashboard/dashboard-enterprise.md) — Veja painéis, widgets e SQL
- [Defeitos - Visão Geral](../defeitos/visao-geral.md) — Veja workflow, sandbox e anexos
