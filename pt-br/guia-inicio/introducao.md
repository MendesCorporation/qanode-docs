# Introdução ao QANode

O **QANode** é uma plataforma de automação de testes de código aberto que permite criar, executar e gerenciar testes automatizados através de um **editor visual de fluxos** baseado em nós. Com uma interface intuitiva de arrastar e conectar, você pode construir cenários de teste complexos sem escrever código — ou combinar a interface visual com JavaScript customizado para máxima flexibilidade.

---

## O que é o QANode?

O QANode foi projetado para equipes de QA, desenvolvedores e engenheiros de teste que precisam de uma ferramenta moderna e visual para automação de testes. A plataforma cobre as principais áreas de teste:

### Automação Web
Teste interfaces de usuário usando o **Playwright**, o framework de automação mais moderno do mercado. O QANode oferece dois nós especializados:

- **Web Flow** — Automação baseada em seletores CSS, XPath, data-testid e texto
- **Smart Locators** — Automação com localizadores semânticos (`getByRole`, `getByLabel`, `getByPlaceholder`, etc.)

Ambos suportam múltiplos navegadores (Chromium, Firefox, WebKit), modo headless, captura de screenshots e reuso de sessão.

### Testes de API
Execute requisições HTTP com suporte completo a:
- Métodos GET, POST, PUT, PATCH e DELETE
- Autenticação Bearer, Basic e API Key
- Headers customizados e corpo da requisição
- Integração com credenciais salvas

### Banco de Dados
Conecte-se e valide dados em:
- **PostgreSQL**, **MySQL**, **MariaDB**, **Oracle** — com query builder visual ou SQL direto
- **MongoDB** — com operações find, insert, update, delete e aggregation pipeline

### Infraestrutura
Execute comandos remotos via **SSH** com suporte a múltiplos passos, autenticação por senha ou chave privada, e captura de saída.

### Extensibilidade
Crie **nós customizados** em qualquer linguagem de programação (Node.js, Python, Java, C#, Go, etc.) através do sistema de provedores HTTP.

---

## Características Principais

| Recurso | Descrição |
|---------|-----------|
| **Editor Visual** | Canvas interativo com arrastar e conectar nós |
| **18+ Nós Nativos** | Controle de fluxo, web, API, banco de dados, infra e utilitários |
| **Nós Customizados** | Extensível com provedores HTTP em qualquer linguagem |
| **Projetos e Suítes** | Organize testes em projetos e agrupe em suítes |
| **Agendamento** | Execute testes automaticamente com expressões cron |
| **Dashboard** | Painéis customizáveis com gráficos e métricas em tempo real |
| **Relatórios PDF** | Geração automática com templates configuráveis |
| **Variáveis e Credenciais** | Gerenciamento centralizado e seguro |
| **Controle de Acesso** | Usuários, papéis e permissões granulares |
| **Auditoria** | Log completo de todas as ações do sistema |
| **Gravador Chrome** | Extensão que grava interações e gera fluxos |
| **Versão Desktop** | Aplicativo standalone com banco de dados embutido |
| **Expressões** | Sistema de interpolação `{{ }}` com acesso a outputs e variáveis |
| **Tempo Real** | Atualizações ao vivo do status de execução |

---

## Arquitetura

O QANode é composto por quatro componentes principais:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Frontend    │    │  API Server  │    │   Worker     │
│  (React)     │◄──►│  (Fastify)   │◄──►│  (Executor)  │
│  Vite + TS   │    │  REST API    │    │  Playwright  │
└─────────────┘    └──────┬───────┘    └─────────────┘
                          │
                   ┌──────▼───────┐
                   │  PostgreSQL   │
                   │  (Prisma ORM) │
                   └──────────────┘
```

- **Frontend** — Interface de usuário em React com TypeScript, construída com Vite
- **API Server** — Servidor REST em Fastify com autenticação JWT
- **Worker** — Motor de execução que processa os fluxos e controla Playwright
- **PostgreSQL** — Banco de dados relacional para armazenamento de dados (via Prisma ORM)

### Versão Desktop

A versão desktop empacota todos os componentes em um único aplicativo **Electron**, incluindo um PostgreSQL embutido. Basta instalar e usar — sem necessidade de configuração de infraestrutura.

---

## Community Edition vs Enterprise

Esta documentação cobre a **Community Edition**, que é gratuita. A tabela abaixo resume as diferenças:

| Recurso | Community | Enterprise |
|---------|-----------|------------|
| Editor de Fluxos | ✅ | ✅ |
| Todos os Nós Nativos | ✅ | ✅ |
| Nós Customizados | ✅ | ✅ |
| Projetos e Suítes | ✅ | ✅ |
| Agendamento | ✅ | ✅ |
| Variáveis e Credenciais | ✅ | ✅ |
| Relatórios PDF | ✅ | ✅ |
| Gravador Chrome | ✅ | ✅ |
| Versão Web | ❌ | ✅ |
| Dashboard Customizável | ❌ | Completo |
| Multi-usuário | ❌ | Completo |
| MFA (Autenticação 2FA) | ❌ | ✅ |
| Rastreamento de Criador | ❌ | ✅ |
| Suporte Dedicado | ❌ | ✅ |

---

## Próximos Passos

- [Instalação](instalacao.md) — Configure o QANode no seu ambiente
- [Início Rápido](inicio-rapido.md) — Crie seu primeiro teste em minutos
- [Conceitos Fundamentais](conceitos.md) — Entenda a estrutura da plataforma
