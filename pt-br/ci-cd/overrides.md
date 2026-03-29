# Overrides por Execução

> **Disponível em:** QANode Enterprise

Overrides permitem alterar valores de variáveis e credenciais apenas durante a execução atual do pipeline, sem modificar o cadastro permanente do QANode.

---

## Quando usar

Overrides são úteis quando a empresa precisa:

- apontar para um ambiente temporário de PR
- trocar a URL base de uma API
- usar um token efêmero do pipeline
- ajustar senha temporária de um ambiente de homologação
- executar a mesma suíte contra múltiplos ambientes sem editar o fluxo

---

## Override de variáveis

Use a flag:

```bash
--var NOME=VALOR
```

Exemplos:

```bash
--var BASE_URL=https://preview.app
--var LOCALE=pt-BR
--var FEATURE_FLAG=true
```

Exemplo completo:

```bash
npx @qanode/cli run scenario \
  --project-name "Checkout" \
  --scenario-name "Login API" \
  --var BASE_URL=https://preview.app \
  --wait
```

---

## Override de credenciais

Use a flag:

```bash
--credential CREDENCIAL.CAMPO=VALOR
```

Exemplos:

```bash
--credential api-main.token=$API_TOKEN
--credential smtp-main.password=$SMTP_PASSWORD
--credential pg-staging.host=db-preview.internal
```

Exemplo completo:

```bash
npx @qanode/cli run scenario \
  --scenario-id SCENARIO_ID \
  --credential api-main.token=$API_TOKEN \
  --credential api-main.baseUrl=https://preview.api \
  --wait
```

---

## Comportamento dos overrides

Overrides:

- valem apenas para a execução atual
- não sobrescrevem o valor salvo no banco
- não transformam a variável ou credencial em um novo cadastro

Em outras palavras: o QANode usa o valor override somente naquele run.

---

## Relação com variáveis e credenciais cadastradas

O valor persistido continua sendo a referência oficial do produto:

- [Variáveis](../variaveis/variaveis.md)
- [Credenciais](../credenciais/credenciais.md)

O override é uma camada temporária acima desse cadastro.

---

## Boas práticas

- Use override apenas para valores que realmente mudam por ambiente ou por pipeline
- Guarde segredos em **Secrets** do seu provedor de CI/CD
- Prefira URLs completas, incluindo `https://`, quando o cenário navegar diretamente para uma página
- Se o alvo for identificado por nome, informe também o projeto

---

## O que não muda automaticamente

Hoje o override é aplicado na execução disparada pelo pipeline. Ele não implica, por si só, que outros fluxos derivados herdem esse mesmo contexto automaticamente.

Por isso, trate o override como parte da execução de CI/CD, não como alteração permanente do cenário.

---

## Próximos Passos

- [Exemplos de Pipeline](./exemplos-pipelines.md) — Veja implementações completas em GitHub Actions e Azure DevOps
