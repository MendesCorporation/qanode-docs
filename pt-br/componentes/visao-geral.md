# Componentes Reutilizáveis

Os **Componentes Reutilizáveis** permitem transformar um trecho de automação em um bloco reaproveitável. Em vez de copiar os mesmos nós de login, preparação de massa, consulta auxiliar ou validação comum em vários cenários, você cria um componente uma vez, define suas entradas e saídas, publica e usa esse componente dentro dos cenários.

Pense em um componente como um **subfluxo com contrato**:

- o componente recebe valores de entrada;
- executa uma sequência própria de nós;
- devolve uma saída para o cenário que o chamou.

---

## Quando Usar

Use componentes quando houver uma sequência que se repete em muitos cenários, por exemplo:

- autenticar em um sistema;
- criar uma massa de dados padrão;
- consultar dados em API ou banco e normalizar o resultado;
- executar uma validação de negócio comum;
- encapsular uma automação web pequena que serve de preparação para outros testes.

Evite componentes para ações muito específicas de um único cenário. Se a lógica só aparece uma vez, manter os nós diretamente no cenário costuma ser mais simples.

---

## Onde Encontrar

No menu lateral, acesse **Componentes**.

A listagem permite:

- criar um novo componente;
- buscar por nome;
- filtrar por categoria;
- filtrar por status **Publicado** ou **Rascunho**;
- copiar o nome ou ID do componente;
- excluir componentes, respeitando as permissões do usuário.

Quando um componente está sendo usado por cenários, o QANode avisa antes da exclusão para evitar remoções acidentais.

---

## Criando um Componente

1. Acesse **Componentes**.
2. Clique em **Novo Componente**.
3. Informe:
   - **Nome**: nome exibido na lista e no editor de cenários;
   - **Categoria**: grupo usado para organizar a paleta de componentes;
   - **Cor**: cor visual do componente no canvas.
4. Clique para criar.

O QANode abre o editor do componente. Ele usa a mesma base visual do editor de fluxos, mas em **modo de componente**.

[Criando um componente](../../assets/images/criar-componente.mp4)

---

## Editor do Componente

Todo componente possui dois nós especiais:

| Nó | Função |
|----|--------|
| **Input** | Define os dados que o cenário precisa enviar para o componente |
| **Output** | Define o valor que o componente devolve ao cenário |

Esses nós são protegidos no editor. Eles fazem parte do contrato do componente e não devem ser removidos.

Entre o **Input** e o **Output**, você adiciona os nós normais do QANode: Web, API, banco de dados, utilitários, controle de fluxo e outros nós disponíveis no seu ambiente.

---

## Entradas do Componente

No nó **Input**, configure os campos que o componente espera receber.

| Campo | Descrição |
|-------|-----------|
| **Nome do Campo** | Nome usado para mapear o valor no cenário |
| **Tipo** | `string`, `number`, `boolean`, `object` ou `array` |
| **Obrigatório** | Define se o cenário precisa preencher esse valor |
| **Dados de Teste** | Valor usado ao testar o componente diretamente no editor |

Exemplo de entradas:

| Nome | Tipo | Obrigatório |
|------|------|-------------|
| `email` | `string` | Sim |
| `senha` | `string` | Sim |
| `perfil` | `string` | Não |

[Definindo entradas de um componente](../../assets/images/entradas-componente.mp4)

Durante a execução do componente, os nós internos podem usar essas entradas por expressão, como qualquer outro output de nó.

Exemplo:

```
{{ steps.Input.outputs.email }}
```

> Dê nomes simples e estáveis para os campos. Isso facilita o uso do componente em cenários e evita expressões difíceis de manter.

---

## Saída do Componente

No nó **Output**, configure o que o componente devolve ao cenário.

| Campo | Descrição |
|-------|-----------|
| **Nome do Campo** | Nome da saída exposta para o cenário |
| **Tipo** | Tipo esperado do valor |
| **Valor** | Valor literal ou expressão calculada a partir dos nós internos |

[Definindo saída um componente](../../assets/images/saida-componente.mp4)

Exemplo:

```
Nome: token
Tipo: string
Valor: {{ steps.login.outputs.json.token }}
```

O cenário que chama o componente poderá usar a saída como output do nó de componente.

> Nesta versão, o componente trabalha com uma saída principal no nó Output. Se precisar retornar mais de um dado, use um objeto como saída, por exemplo `{ token, userId, role }`.

---

## Testando o Componente

Antes de publicar, teste o componente no próprio editor.

1. Preencha **Dados de Teste** nos campos obrigatórios do nó Input.
2. Clique em **Testar Componente**.
3. Acompanhe a execução no painel de runs do editor.

As execuções de teste servem para validar o componente isoladamente. Elas não substituem as execuções oficiais dos cenários que usarão o componente.

Se um campo obrigatório não tiver dado de teste, o QANode bloqueia a execução do teste do componente até o valor ser informado.

[Testando um componente](../../assets/images/testar-component.mp4)

---

## Salvando e Publicando

Ao salvar o componente, o QANode registra:

- o fluxo interno do componente;
- o contrato de entrada;
- o contrato de saída;
- nome, categoria e cor.

Para que o componente apareça na paleta dos cenários, ele precisa estar **Publicado**.

Regras importantes:

- componentes em rascunho não aparecem para uso em cenários;
- ao alterar o fluxo interno, entradas ou saída, o componente volta a exigir revisão/publicação;
- publicar confirma que o contrato atual está pronto para ser usado por outros cenários.

[Salvando e publicando um componente](../../assets/images/salvar-e-publicar.mp4)

---

## Histórico de Versões — Enterprise

No QANode Enterprise, componentes publicados podem manter histórico de versões.

Cada publicação relevante pode gerar um snapshot com o fluxo interno, contrato de entrada, contrato de saída e metadados do componente. Isso permite consultar versões anteriores e restaurar uma versão quando necessário.

Como usar:

1. Abra o componente.
2. Acesse o **Histórico de Versões**.
3. Selecione uma versão para visualizar em modo de consulta.
4. Use **Restaurar esta versão** quando quiser voltar ao contrato e ao fluxo daquele snapshot.

> A quantidade de versões mantidas por componente é definida pelo Super Admin em **Configurações → Geral**.

---

## Usando em um Cenário

No editor de cenários:

1. Abra a aba **Componentes** na paleta lateral.
2. Localize o componente publicado.
3. Arraste o componente para o canvas.
4. Conecte-o aos outros nós.
5. Preencha os campos de entrada no painel de propriedades.

As entradas aceitam valores literais e expressões:

```
email: {{ variables.USUARIO_TESTE }}
senha: {{ variables.SENHA_TESTE }}
perfil: admin
```

Durante a execução, o QANode executa o componente como parte do cenário. Se uma entrada obrigatória estiver vazia, o cenário é bloqueado antes de rodar para evitar falha difícil de diagnosticar.

[Usando um componente em cenário](../../assets/images/usar-em-cenario.mp4)

---

## Usando Outputs do Componente

Depois que o componente roda, sua saída fica disponível como output do nó no cenário.

Exemplo:

```
{{ steps.loginComponente.outputs.token }}
```

Se a saída principal for um objeto:

```
{{ steps.prepararUsuario.outputs.result.userId }}
{{ steps.prepararUsuario.outputs.result.email }}
```

[Uso de output em componente](../../assets/images/usar-output-component.mp4)

---

## Permissões

Em ambientes com controle de acesso, os componentes respeitam permissões próprias:

| Permissão | O que permite |
|-----------|---------------|
| `component.view` | Ver a lista e abrir componentes |
| `component.create` | Criar e editar componentes próprios |
| `component.edit` | Editar componentes de outros usuários |
| `component.delete.own` | Excluir componentes próprios |
| `component.delete` | Excluir qualquer componente |
| `component.publish` | Publicar componentes |

---

## Boas Práticas

- Use componentes para lógicas realmente compartilhadas.
- Mantenha o contrato pequeno e claro.
- Nomeie entradas e saídas com termos de negócio, não detalhes técnicos internos.
- Use categorias para organizar componentes por domínio, como `Login`, `Massa de dados`, `Pedidos` ou `CRM`.
- Teste o componente isoladamente antes de publicar.
- Ao alterar um componente usado por muitos cenários, valide pelo menos um cenário consumidor antes de liberar para o time.
- Prefira retornar um objeto quando a saída precisa carregar vários valores relacionados.

---

## Limitações e Cuidados

- Componentes publicados podem ser usados por vários cenários; mudanças no contrato podem exigir ajuste nos cenários consumidores.
- Componentes em rascunho não aparecem na paleta do editor de cenários.
- O nó Input e o nó Output são parte da estrutura do componente e não devem ser tratados como nós comuns.
- Se o componente depende de sessão web, credenciais ou variáveis, documente isso no nome, categoria ou no padrão de uso do time.
