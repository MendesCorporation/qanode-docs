# Nó Gerador de Dados

O nó **Gerador de Dados** cria massas sintéticas para testes sem depender de serviços externos. Em uma única execução, ele pode gerar de 1 a 1.000 registros com dados de identidade, documentos, endereços, empresas, internet, textos, números, datas, cartões e comércio.

Use este nó quando um cenário precisar de usuários, pedidos, produtos, documentos ou outros valores novos a cada execução. Os campos configurados ficam disponíveis no seletor de variáveis e podem ser usados diretamente pelos nós seguintes.

> Os dados são sintéticos. Um documento estruturalmente válido respeita o formato e os dígitos verificadores aplicáveis, mas não representa necessariamente uma pessoa, empresa, endereço, telefone ou meio de pagamento real.

---

## Configuração Geral

| Campo | Descrição |
|-------|-----------|
| **Idioma e formato regional** | Define o idioma e o padrão regional usados na geração |
| **País padrão** | Define o país usado por documentos, telefones, endereços e empresas quando o campo não possui uma configuração própria |
| **Quantidade** | Número de registros gerados, entre 1 e 1.000 |
| **Caracteres** | Mantém caracteres naturais ou normaliza textos para ASCII, removendo acentos |
| **Semente** | Valor opcional que permite reproduzir os mesmos dados com a mesma configuração |
| **Campos gerados** | Lista de propriedades que formarão cada registro |

Formatos regionais disponíveis:

- português do Brasil e de Portugal;
- inglês dos Estados Unidos, Reino Unido, Canadá e Austrália;
- espanhol, francês, alemão e italiano;
- japonês e chinês simplificado.

---

## Adicionando Campos

Clique em **Adicionar campo**, escolha o tipo de dado e informe um **Nome da saída** único. Esse nome será a chave usada nas expressões dos próximos nós.

Os campos são organizados nas seguintes categorias:

| Categoria | Exemplos |
|-----------|----------|
| **Identidade e dados pessoais** | nome, sobrenome, nome completo, e-mail, usuário, senha, nascimento, idade, telefone e avatar fictício |
| **Documentos e cartões** | documento pessoal, documento empresarial, passaporte, número, bandeira, CVV e validade do cartão |
| **Endereço e localização** | endereço completo, logradouro, número, bairro, cidade, estado, código postal, país e coordenadas |
| **Empresa** | razão social, nome fantasia, departamento, cargo, domínio, e-mail e telefone corporativo |
| **Internet e desenvolvimento** | UUID, URL, domínio, IPv4, IPv6, MAC address, hash, token, slug, cor hexadecimal e user agent |
| **Textos, números e valores** | palavra, frase, parágrafo, inteiro, decimal, valor monetário, percentual, booleano, datas, lista, sequência, string e template |
| **Comércio** | produto, SKU, preço, quantidade, moeda, número do pedido, rastreamento e status do pedido |

As opções exibidas dependem do tipo selecionado. Entre elas estão país, formato, validade estrutural, bandeira do cartão, máscara, tamanho, prefixo, sufixo, domínio, moeda, casas decimais, intervalos numéricos e de data, lista de valores e composição de senha.

### Registros coerentes

Campos relacionados compartilham o mesmo contexto dentro de cada registro. Por exemplo:

- nome, sobrenome, nome completo, usuário e e-mail pertencem à mesma pessoa sintética;
- cidade, estado, código postal e endereço completo pertencem ao mesmo endereço sintético;
- número, bandeira, CVV e validade pertencem ao mesmo cartão sintético;
- produto, SKU, preço, moeda, pedido e rastreamento pertencem ao mesmo contexto comercial.

Essa coerência vale dentro de cada item. Ao gerar vários registros, cada item recebe seu próprio contexto.

---

## Dados Válidos e Inválidos

Campos compatíveis permitem escolher entre **Estruturalmente válido** e **Estruturalmente inválido**. O modo inválido é útil para validar rejeições, mensagens de erro e regras de entrada.

Essa opção está disponível para documentos pessoais e empresariais, passaportes, telefones, números de cartão e máscaras personalizadas.

O modo inválido altera a estrutura do valor deliberadamente. Ele não consulta bases governamentais, operadoras ou emissores.

---

## Máscaras, Sequências e Templates

Máscaras personalizadas usam os seguintes caracteres:

| Caractere | Resultado |
|-----------|-----------|
| `#` | Dígito |
| `A` | Letra maiúscula |
| `a` | Letra minúscula |
| `*` | Caractere alfanumérico |

Exemplo: `PED-####-AA`.

Uma **Sequência** pode definir início, incremento e preenchimento com zeros. Use-a para criar valores como `0001`, `0002` e `0003`.

Um **Template** combina valores do registro com texto fixo, sequência e máscara. Exemplo:

```text
USR-{sequence}-{firstName}-###
```

Os valores inseridos, como `{firstName}`, são preservados literalmente. Caracteres como `A`, `a`, `*` ou `#` existentes no nome não são reinterpretados como máscara.

---

## Saídas

| Saída | Descrição |
|-------|-----------|
| `data` | Array com todos os registros gerados |
| `count` | Quantidade de registros gerados |
| `first` | Primeiro registro gerado |

Se o nó estiver rotulado como **Gerar usuário** e tiver campos chamados `nome` e `email`, use:

```text
{{ steps["Gerar usuário"].outputs.first.nome }}
{{ steps["Gerar usuário"].outputs.first.email }}
{{ steps["Gerar usuário"].outputs.count }}
{{ steps["Gerar usuário"].outputs.data }}
```

Para processar vários registros, passe `outputs.data` para um nó [Loop](../controle-fluxo/loop.md).

---

## Geração Assistida por IA

Uma IA conectada ao QANode por MCP pode consultar o contrato do Gerador de Dados, criar sua configuração e usar os nomes de saída nos passos seguintes. Descreva a massa pelo objetivo do teste, por exemplo:

```text
Crie dez clientes brasileiros com nome, e-mail no domínio exemplo.com,
CPF válido, telefone internacional e número de pedido sequencial.
```

Revise os nomes das saídas e as regras de validade antes de executar, especialmente quando a massa será usada em testes negativos.

---

## Boas Práticas

- Use uma semente quando precisar reproduzir uma falha com os mesmos dados.
- Use nomes de saída curtos e únicos, como `email_cliente` e `cpf_invalido`.
- Gere apenas os campos usados pelo cenário para manter as saídas fáceis de revisar.
- Use ASCII quando o sistema testado não aceita acentos ou caracteres não latinos.
- Não utilize os dados gerados como dados reais, credenciais reais ou informações de pagamento.
