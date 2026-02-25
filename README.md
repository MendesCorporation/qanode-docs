# QANode Documentation

Documentação oficial do **QANode** — Plataforma de orquestração de testes com editor visual de fluxos.

## Idiomas Disponíveis / Available Languages / Idiomas Disponibles

| Idioma | Status |
|--------|--------|
| [Português Brasileiro](pt-br/README.md) | ✅ Disponível |
| [English](en/README.md) | ✅ Available |
| [Español](es/README.md) | ✅ Disponible |

## Sobre o QANode

O QANode é uma plataforma completa de automação e orquestração de testes que permite criar, executar e gerenciar testes automatizados através de um editor visual de fluxos baseado em nós (nodes). A plataforma suporta testes web com Playwright, requisições HTTP/API, consultas a bancos de dados, comandos SSH e muito mais.

### Principais Características

- **Editor Visual de Fluxos** — Crie testes arrastando e conectando nós
- **Automação Web** — Testes de interface com Playwright (Chromium)
- **Smart Locators** — Localizadores semânticos baseados em ARIA roles e acessibilidade
- **Testes de API** — Requisições HTTP com suporte a autenticação
- **Banco de Dados** — PostgreSQL, MySQL, MariaDB, Oracle, MongoDB
- **Infraestrutura** — Comandos SSH remotos
- **Nós Customizados** — Extensível com provedores HTTP em qualquer linguagem
- **Dashboard** — Painéis customizáveis com gráficos e métricas
- **Relatórios PDF** — Geração automática com templates configuráveis
- **Agendamento** — Execução programada com expressões cron
- **Gravador Chrome** — Extensão que grava interações e gera fluxos automaticamente

## Estrutura da Documentação

```
docs/
├── README.md                    # Este arquivo
├── assets/
│   └── images/                  # Imagens e capturas de tela
├── pt-br/                       # Documentação em Português
│   ├── README.md                # Índice principal
│   ├── guia-inicio/             # Introdução e primeiros passos
│   ├── editor-fluxos/           # Editor visual de fluxos
│   ├── nos/                     # Referência de nós (nodes)
│   │   ├── controle-fluxo/      # If, Switch, Loop, Merge
│   │   ├── web/                 # Web Flow, Smart Locators
│   │   ├── api/                 # HTTP Request
│   │   ├── banco-dados/         # PostgreSQL, MySQL, etc.
│   │   ├── infraestrutura/      # SSH Command
│   │   └── utilitarios/         # Set Variable, Log, Wait, etc.
│   ├── nos-customizados/        # Criando nós customizados
│   ├── projetos/                # Gerenciamento de projetos
│   ├── suites/                  # Suítes de teste e agendamento
│   ├── variaveis/               # Variáveis globais
│   ├── credenciais/             # Gerenciamento de credenciais
│   ├── dashboard/               # Painéis e widgets
│   ├── relatorios/              # Relatórios e exportação
│   ├── expressoes/              # Sistema de expressões {{ }}
│   ├── administracao/           # Usuários, papéis, configurações
│   └── desktop/                 # Versão desktop (Electron)
├── en/                          # English documentation
└── es/                          # Documentación en Español
```

## Licença

Esta documentação está sob a mesma licença da empresa Apptrix LTDA.
