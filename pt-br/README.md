# QANode — Documentação

Bem-vindo à documentação oficial do **QANode Community Edition**, a plataforma de automação de testes com editor visual de fluxos.

---

## Primeiros Passos

| Página | Descrição |
|--------|-----------|
| [Introdução](guia-inicio/introducao.md) | O que é o QANode e visão geral da plataforma |
| [Instalação](guia-inicio/instalacao.md) | Como instalar e configurar o QANode |
| [Início Rápido](guia-inicio/inicio-rapido.md) | Crie seu primeiro teste em minutos |
| [Conceitos Fundamentais](guia-inicio/conceitos.md) | Projetos, fluxos, nós, suítes e execuções |

---

## Editor de Fluxos

| Página | Descrição |
|--------|-----------|
| [Visão Geral do Editor](editor-fluxos/visao-geral.md) | Interface, canvas, paleta de nós e painel de propriedades |
| [Trabalhando com Nós](editor-fluxos/trabalhando-com-nos.md) | Adicionar, conectar, configurar e remover nós |
| [Execução e Depuração](editor-fluxos/execucao-depuracao.md) | Executar fluxos, visualizar resultados e depurar falhas |

---

## Referência de Nós

### Controle de Fluxo
| Nó | Descrição |
|----|-----------|
| [If](nos/controle-fluxo/if.md) | Desvio condicional (verdadeiro/falso) |
| [Switch](nos/controle-fluxo/switch.md) | Desvio múltiplo por valor ou condição |
| [Loop](nos/controle-fluxo/loop.md) | Repetição por contagem, array ou condição |
| [Merge](nos/controle-fluxo/merge.md) | Junção de múltiplos caminhos |

### Web
| Nó | Descrição |
|----|-----------|
| [Web Flow](nos/web/web-flow.md) | Automação web com múltiplos passos e seletores CSS/XPath |
| [Smart Locators](nos/web/smart-locators.md) | Automação web com localizadores semânticos do Playwright |

### API
| Nó | Descrição |
|----|-----------|
| [HTTP Request](nos/api/http-request.md) | Requisições HTTP (GET, POST, PUT, PATCH, DELETE) |

### Banco de Dados
| Nó | Descrição |
|----|-----------|
| [PostgreSQL](nos/banco-dados/postgresql.md) | Consultas e operações no PostgreSQL |
| [MySQL](nos/banco-dados/mysql.md) | Consultas e operações no MySQL |
| [MariaDB](nos/banco-dados/mariadb.md) | Consultas e operações no MariaDB |
| [Oracle](nos/banco-dados/oracle.md) | Consultas e operações no Oracle |
| [MongoDB](nos/banco-dados/mongodb.md) | Operações no MongoDB (find, insert, update, etc.) |

### Infraestrutura
| Nó | Descrição |
|----|-----------|
| [SSH Command](nos/infraestrutura/ssh-command.md) | Execução de comandos SSH remotos |

### Utilitários
| Nó | Descrição |
|----|-----------|
| [Set Variable](nos/utilitarios/set-variable.md) | Define variáveis em tempo de execução |
| [Log](nos/utilitarios/log.md) | Registra mensagens no log de execução |
| [Wait](nos/utilitarios/wait.md) | Aguarda por tempo ou condição |
| [Stop and Fail](nos/utilitarios/stop-and-fail.md) | Interrompe o fluxo com status de falha |
| [Custom JavaScript](nos/utilitarios/custom-javascript.md) | Executa código JavaScript customizado |

---

## Nós Customizados

| Página | Descrição |
|--------|-----------|
| [Visão Geral](nos-customizados/visao-geral.md) | Como funciona o sistema de provedores |
| [Criando um Provedor - Enterprise](nos-customizados/criando-provedor-enterprise.md) | Passo a passo para criar um provedor HTTP |
| [Contrato da API](nos-customizados/contrato-api.md) | Endpoints obrigatórios e formato de dados |
| [Exemplos](nos-customizados/exemplos.md) | Exemplos em Node.js, Python, Java, C# e Go |
| [Desktop: Nós Locais](nos-customizados/nos-locais-desktop.md) | Criando nós locais na versão desktop |
| [QANode.MD (IA)](../QANode.MD) | Guia para IA criar nós e diagnosticar problemas |

---

## Gerenciamento

| Página | Descrição |
|--------|-----------|
| [Projetos](projetos/projetos.md) | Criação e gerenciamento de projetos de teste |
| [Suítes de Teste](suites/suites.md) | Agrupamento de fluxos e agendamento |
| [Variáveis](variaveis/variaveis.md) | Variáveis globais e secretas |
| [Credenciais](credenciais/credenciais.md) | Gerenciamento seguro de credenciais |

---

## Monitoramento e Relatórios

| Página | Descrição |
|--------|-----------|
| [Dashboard - Enterprise](dashboard/dashboard-enterprise.md) | Painéis, widgets e gráficos |
| [Relatórios - Enterprise](relatorios/relatorios-enterprise.md) | Geração de relatórios PDF e envio por e-mail |

---

## Referência

| Página | Descrição |
|--------|-----------|
| [Expressões](expressoes/expressoes.md) | Sistema de expressões `{{ }}` e interpolação |
| [Administração - Enterprise](administracao/administracao-enterprise.md) | Usuários, papéis, permissões, SMTP e auditoria |
| [Versão Desktop](desktop/desktop.md) | Instalação e recursos exclusivos da versão desktop |
| [Extensão Chrome](desktop/extensao-chrome.md) | Gravador de testes para o navegador |
