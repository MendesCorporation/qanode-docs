# Tokens de Integração

> **Disponível em:** QANode Enterprise

Tokens de integração são credenciais próprias para automação. Eles permitem que pipelines, scripts e clientes MCP chamem o QANode sem depender de login no navegador.

O mesmo token pode ser usado para:

- **CI/CD** — executar cenários, suítes, consultar runs e baixar relatórios por CLI/API;
- **MCP** — permitir que uma IA opere o QANode respeitando as permissões do usuário autenticado.

---

## Onde acessar

Vá em:

**Configurações → Tokens de Acesso**

Essa tela só aparece para usuários com as permissões adequadas.

---

## Permissões

| Permissão | O que permite |
|-----------|---------------|
| `settings.integration_token` | Criar e revogar os próprios tokens |
| `settings.integration_token_all` | Ver todos os tokens, revogar tokens de outras pessoas e controlar a política global de expiração |

Na configuração padrão do produto:

- **Super Admin** e **Admin** costumam ter acesso completo
- **Architect** pode ter acesso à criação dos próprios tokens, conforme o papel configurado

> **Importante:** O token herda a capacidade operacional do usuário que o utiliza. Ou seja, ter um token não concede acesso extra; ele apenas autentica de forma não interativa.

---

## Criando um token

[Criando token](../../assets/images/ci-cd-criando-access-token.mp4)

1. Acesse **Configurações → Tokens de Acesso**
2. Clique em **Gerar Token**
3. Defina:
   - **Nome** do token
   - **Prazo de expiração**, quando a política global permitir escolha individual
4. Copie o valor gerado
5. Salve esse valor em um **secret** do seu pipeline, da IDE ou do cliente MCP

O token é exibido integralmente apenas no momento da criação. Depois disso, a interface mantém apenas informações de referência e auditoria.

---

## Prefixo do token

Tokens de integração do QANode usam o prefixo:

```bash
qnt_
```

Esse prefixo facilita a identificação em logs, variáveis de ambiente e cofres de segredo.

---

## Política global de expiração

Usuários com `settings.integration_token_all` podem configurar a política global em:

**Configurações → Tokens de Acesso → Política Global de Expiração de Tokens**

As opções são:

- **Cada usuário escolhe** — a pessoa define se o token expira e em quanto tempo
- **Expiração fixa** — todo token novo expira após o mesmo prazo definido pela administração

Quando a política global for fixa, o usuário deixa de escolher o prazo individual no modal de criação.

---

## Revogação

Revogar um token interrompe imediatamente o uso dele em novos comandos CLI, chamadas API ou conexões MCP.

Você pode revogar:

- seus próprios tokens, com `settings.integration_token`
- qualquer token da instância, com `settings.integration_token_all`

---

## Boas práticas de segurança

- Guarde `QANODE_TOKEN` em **Secrets** do provedor de CI/CD
- Para MCP, guarde o token no cofre/configuração segura da IDE ou do cliente de IA
- Nunca escreva tokens em arquivos versionados
- Prefira um token por pipeline, por cliente MCP ou por sistema integrado
- Revogue tokens que não são mais usados
- Use expiração quando o processo da sua empresa exigir rotação periódica

---

## Exemplo de uso com secret

```bash
export QANODE_URL=https://qanode.empresa.com
export QANODE_TOKEN=qnt_xxxxx
```

No GitHub Actions ou Azure DevOps, o valor normalmente fica em um secret do job e não em texto puro no repositório.

---

## Exemplo de uso com MCP

Clientes MCP usam a URL da API do QANode e enviam o token no header `Authorization`.

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

> Para detalhes de uso com IA, veja [MCP — Integração com IA](../mcp/visao-geral.md).

---

## Auditoria

O QANode registra eventos relacionados a tokens de integração, como:

- criação
- revogação
- atualização de política global de expiração

Isso ajuda a rastrear quem criou ou revogou cada chave operacional.

---

## Próximos Passos

- [CLI e API do CI/CD](./cli-api.md) — Veja como autenticar, disparar execuções e consultar resultados
- [MCP — Integração com IA](../mcp/visao-geral.md) — Veja como conectar uma IA ao QANode
