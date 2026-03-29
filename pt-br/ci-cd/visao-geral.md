# Integração CI/CD — Visão Geral

> **Disponível em:** QANode Enterprise

O módulo de integração CI/CD do QANode permite disparar cenários e suítes a partir do pipeline da sua empresa, autenticar com tokens de integração e baixar o relatório consolidado de cada execução sem depender de sessão de usuário no navegador.

Em vez de tratar o QANode apenas como uma interface visual, sua empresa pode usá-lo como parte oficial do processo de validação de build, pull request, release ou deploy.

---

## O que a integração oferece

- **Tokens de integração** para autenticação em pipelines e automações
- **CLI oficial (`@qanode/cli`)** para uso em GitHub Actions, Azure DevOps, GitLab, Jenkins e scripts locais
- **Rotas `/api/ci` dedicadas** para cenários, suítes, consulta de execução e download de relatório
- **Execução por ID ou por nome** com suporte a projeto quando necessário
- **Overrides por execução** para variáveis e credenciais
- **Saída simplificada para pipeline** com `success`, `failed` ou `cancelled`
- **Relatório consolidado em PDF** para anexar ao job do pipeline

---

## Como funciona em alto nível

O fluxo operacional mais comum é:

1. O time cria um **token de integração** em **Configurações → Access Tokens**
2. O pipeline salva esse token como **secret**
3. O job executa a CLI do QANode com `QANODE_URL` e `QANODE_TOKEN`
4. O QANode cria uma execução real na instância
5. O pipeline espera o resultado final e decide se continua ou falha
6. Opcionalmente, o job baixa o **relatório PDF consolidado**

---

## Conceitos principais

### URL pública do QANode

A variável `QANODE_URL` deve apontar para a **URL pública** que o time já usa para abrir o QANode no navegador.

Na maioria dos ambientes corporativos isso significa:

- o mesmo domínio público do frontend
- com proxy ou balanceador encaminhando `/api` para a API do QANode

Exemplos:

```bash
https://qanode.empresa.com
https://qa.empresa.com/qanode
```

Em ambiente local de desenvolvimento, `http://localhost:3000` também pode funcionar quando o frontend estiver proxyando `/api`.

### Token de integração

É um token gerado especificamente para automação. Ele substitui a necessidade de login humano no pipeline e respeita as permissões do usuário ou papel que o criou.

### Cenário

É uma automação individual do QANode, composta pelo fluxo de passos que valida um comportamento específico da aplicação. No CI/CD, você pode executar um cenário isolado quando quiser validar um caminho crítico sem rodar a suíte completa.

### Suíte

A suíte agrupa vários cenários e é o recurso mais comum para regressão completa, smoke tests e gates de release.

### Override por execução

Overrides permitem alterar variáveis ou campos de credenciais **somente para aquela execução do pipeline**, sem sobrescrever o valor persistido no QANode.

---

## Permissões envolvidas

As permissões principais da integração CI/CD são:

| Permissão | O que permite |
|-----------|---------------|
| `settings.integration_token` | Criar e revogar os próprios tokens de integração |
| `settings.integration_token_all` | Ver todos os tokens, revogar tokens de outras pessoas e definir a política global de expiração |
| `flow.run` | Executar cenários via CLI/API |
| `suite.run` | Executar suítes via CLI/API |
| `run.view` / `run.view.all` | Consultar execuções e baixar o relatório |

As permissões de tokens são configuradas em **Configurações → Papéis**.

---

## O que a integração não faz automaticamente

Alguns comportamentos são importantes para evitar expectativa errada:

- a CLI **não descobre o token sozinha**; ele deve ser fornecido pelo pipeline
- a CLI **não precisa de sessão de navegador**
- a execução disparada pelo pipeline é uma **run real** do QANode
- overrides de variáveis e credenciais **não alteram permanentemente** os valores salvos
- o pipeline deve reutilizar o **mesmo ID da run** para baixar o relatório, em vez de disparar uma segunda execução

---

## Navegação desta seção

| Página | O que você vai aprender |
|--------|-------------------------|
| [Tokens de Integração](./tokens-integracao.md) | Como gerar, revogar e governar tokens de pipeline |
| [CLI e API do CI/CD](./cli-api.md) | Como autenticar, disparar cenários/suítes e usar overrides |
| [Exemplos de Pipeline](./exemplos-pipelines.md) | Como integrar com GitHub Actions e Azure DevOps |
| [Overrides por Execução](./overrides.md) | Como sobrescrever variáveis e credenciais com segurança |

---

## Próximos Passos

- [Tokens de Integração](./tokens-integracao.md) — Aprenda a gerar, revogar e governar as chaves usadas pelo pipeline
