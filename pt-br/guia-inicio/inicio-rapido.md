# Início Rápido

Neste guia, você criará seu primeiro teste automatizado no QANode em poucos minutos. Vamos criar um teste web simples que navega até uma página, preenche um formulário e verifica o resultado.

---

## Passo 1: Crie um Projeto

1. No menu lateral, clique em **Projetos**
2. Clique no botão **+ Novo Projeto**
3. Preencha:
   - **Nome**: `Meu Primeiro Projeto`
   - **Status**: Ativo
4. Clique em **Criar**

<video
  autoplay
  loop
  muted
  playsinline
  width="100%"
  height="100%"
>
  <source src="../../assets/images/criar-projeto.mp4" type="video/mp4">
</video>

<p><em>Modal de criação de projeto com campos de nome e status</em></p>


---

## Passo 2: Crie um Fluxo (Cenário de Teste)

1. Dentro do projeto, clique na aba **Cenários**
2. Clique em **+ Novo Cenário**
3. Dê o nome: `Login com sucesso`
4. Você será levado ao **Editor de Fluxos**

<!-- ![Editor de fluxos vazio](../../assets/images/editor-vazio.png) -->
*Imagem: Editor de fluxos com o canvas vazio e a paleta de nós à esquerda*

---

## Passo 3: Adicione os Nós

### Adicionando um nó Web Flow

1. Na paleta de nós à esquerda, localize a categoria **Web**
2. Arraste o nó **Smart Locators** para o canvas
3. Clique no nó para abrir o painel de propriedades à direita

### Configurando os Passos

Vamos usar o site de prática **the-internet.herokuapp.com**, uma página pública feita para testes de automação. A própria página exibe as credenciais: usuário `tomsmith` e senha `SuperSecretPassword!`.

No painel de propriedades, configure os seguintes passos:

#### Passo 1: Navegar para a página de login

1. Clique em **+ Adicionar Passo**
2. Selecione a ação **Navigate**
3. No campo **URL**, digite: `https://the-internet.herokuapp.com/login`

#### Passo 2: Preencher o campo de usuário

1. Clique em **+ Adicionar Passo**
2. Selecione a ação **Fill**
3. Configure o localizador:
   - **Método**: `getByLabel`
   - **Valor**: `Username`
4. No campo **Texto**, digite: `tomsmith`

#### Passo 3: Preencher a senha

1. Clique em **+ Adicionar Passo**
2. Selecione a ação **Fill**
3. Configure o localizador:
   - **Método**: `getByLabel`
   - **Valor**: `Password`
4. No campo **Texto**, digite: `SuperSecretPassword!`

#### Passo 4: Clicar no botão de login

1. Clique em **+ Adicionar Passo**
2. Selecione a ação **Click**
3. Configure o localizador:
   - **Método**: `getByRole`
   - **Valor**: `button`
   - **Nome**: `Login`

#### Passo 5: Verificar o resultado

1. Clique em **+ Adicionar Passo**
2. Selecione a ação **Assert**
3. Configure:
   - **Modo**: `textContains`
   - **Localizador**: `getByText` → `You logged into a secure area!`
   - **Texto Esperado**: `You logged into a secure area!`

<!-- ![Passos configurados](../../assets/images/passos-smart-locators.png) -->
*Imagem: Painel de propriedades do Smart Locators com 5 passos configurados*

---

## Passo 4: Salve e Execute

1. Clique no botão **Salvar** (ícone de disquete) na barra superior
2. Clique no botão **Executar** (ícone de play) ▶️

O QANode irá:
1. Abrir um navegador (modo headless por padrão)
2. Navegar até `https://the-internet.herokuapp.com/login`
3. Preencher o campo Username com `tomsmith`
4. Preencher o campo Password com `SuperSecretPassword!`
5. Clicar no botão Login
6. Verificar se a mensagem `You logged into a secure area!` aparece na página

### Resultado da Execução

Após a execução, cada passo mostrará seu status:

- ✅ **Verde**: Passo executado com sucesso
- ❌ **Vermelho**: Passo falhou
- ⏭️ **Cinza**: Passo não executado (pulado)

<!-- ![Resultado da execução](../../assets/images/resultado-execucao.png) -->
*Imagem: Editor mostrando os nós com indicadores de sucesso/falha e o painel de resultados*

Clique em qualquer nó para ver detalhes:
- **Logs**: Mensagens de cada passo executado
- **Outputs**: Dados extraídos (textos, valores, etc.)
- **Screenshots**: Capturas de tela (se configuradas)
- **Duração**: Tempo de execução de cada passo

---

## Passo 5: Adicione Screenshots (Evidências)

Para capturar evidências durante a execução:

1. Expanda qualquer passo no painel de propriedades
2. Na seção **Evidência**, ative **Capturar Screenshot**
3. Selecione o modo:
   - **Antes**: Captura antes de executar o passo
   - **Depois**: Captura depois de executar o passo
   - **Ambos**: Captura antes e depois

As screenshots serão armazenadas como artefatos da execução e incluídas nos relatórios PDF.

---

## Próximos Passos

Parabéns! Você criou e executou seu primeiro teste automatizado. Agora explore:

- [Conceitos Fundamentais](conceitos.md) — Entenda melhor a estrutura do QANode
- [Editor de Fluxos](../editor-fluxos/visao-geral.md) — Domine o editor visual
- [Referência de Nós](../nos/controle-fluxo/if.md) — Conheça todos os nós disponíveis
- [Expressões](../expressoes/expressoes.md) — Use dados dinâmicos nos seus testes
- [Suítes de Teste](../suites/suites.md) — Agrupe e agende seus testes

---

## Exemplo com API

Quer testar uma API? Veja um exemplo rápido usando a **JSONPlaceholder**, uma API REST pública e gratuita para testes:

1. Arraste um nó **HTTP Request** para o canvas
2. Configure:
   - **Método**: `GET`
   - **URL**: `https://jsonplaceholder.typicode.com/users/1`
3. Arraste um nó **If** e conecte à saída do HTTP Request
4. Configure a condição: `{{ steps['HTTP Request'].outputs.status }} === 200`
5. Na saída **true**, adicione um nó **Log** com a mensagem: `Usuário encontrado: {{ steps['HTTP Request'].outputs.json.name }}`

Esse fluxo faz uma requisição GET à JSONPlaceholder e verifica se o status é 200. Se positivo, exibe o nome do usuário retornado (`Leanne Graham`).
