# SSO e Provisionamento de Usuários

O QANode permite centralizar a autenticação em um provedor de identidade (IdP) usando **OpenID Connect (OIDC)** ou **SAML 2.0**. O ciclo de vida dos usuários pode ser feito por **JIT** (criação no primeiro acesso) ou por **SCIM 2.0** (criação, ativação e desativação enviadas pelo IdP).

Este guia explica o que é controlado pelo IdP, o que continua sendo controlado pelo QANode e como configurar os provedores mais comuns.

> **Disponibilidade:** a configuração é exclusiva do Super Admin e fica em **Configurações → Geral → Login único**.

---

## Visão Geral

SSO e SCIM resolvem problemas diferentes:

| Recurso | Responsabilidade | O que faz |
|---------|------------------|-----------|
| **OIDC ou SAML** | Autenticação | Confirma a identidade e permite entrar no QANode com a conta corporativa |
| **JIT** | Provisionamento no login | Cria o usuário no QANode quando ele entra pela primeira vez |
| **SCIM** | Ciclo de vida | Cria, ativa e desativa usuários no QANode a partir do IdP |
| **Papéis do QANode** | Autorização | Define quais projetos, telas e ações o usuário pode acessar |

Na versão atual, o IdP **não controla os papéis nem as permissões**. O QANode recebe a identidade e o estado do usuário, mas continua sendo a fonte de verdade para autorização.

```text
IdP (autenticação e ciclo de vida)
       │
       ├── OIDC ou SAML ──> identidade e login
       └── SCIM ──────────> criar, ativar e desativar
                                  │
                                  ▼
QANode (papel, permissões, projetos e recursos)
```

### Qual combinação escolher

| Cenário | Recomendação |
|---------|-------------|
| O provedor suporta OIDC | Use **OIDC** para autenticação |
| A organização exige SAML | Use **SAML 2.0** |
| O provedor suporta SCIM | Use **SCIM** para provisionamento |
| O provedor não possui SCIM | Use **JIT** e um intervalo curto de revalidação |
| Desativação rápida é requisito obrigatório | Use **SCIM**; JIT sozinho não sincroniza a desativação para o cadastro local |

---

## Antes de Começar

Verifique estes pontos antes de configurar o provedor:

1. O QANode deve estar publicado em uma URL **HTTPS** acessível pelo IdP. Não use `localhost` em produção.
2. Confirme que o Super Admin sabe a senha local e que seu e-mail no QANode é exatamente o e-mail retornado pelo IdP.
3. Escolha um **papel padrão** de baixo privilégio para novos usuários. O papel Super Admin nunca fica disponível como padrão.
4. Defina se o provisionamento será por **JIT** ou **SCIM**.
5. No IdP, permita acesso ao aplicativo apenas para os usuários ou grupos que realmente devem entrar no QANode.
6. Guarde o segredo OIDC e o token SCIM em um cofre de segredos.
7. Faça o teste com uma sessão de Super Admin antes de ativar o SSO para toda a instância.

> A URL pública usada pelo QANode gera as URLs de callback, ACS e metadata. Se domínio, protocolo ou porta estiverem incorretos, o provedor recusará o retorno.

---

## Configuração Geral no QANode

Em **Configurações → Geral → Login único**, preencha os campos comuns:

| Campo | Descrição |
|-------|-----------|
| **Protocolo** | `OpenID Connect (OIDC)` ou `SAML 2.0` |
| **Nome do provedor** | Nome mostrado aos administradores, por exemplo `Microsoft Entra` ou `Google Workspace` |
| **Provisionamento de usuários** | `JIT — Criar no primeiro acesso` ou `SCIM — Gerenciado pelo IdP` |
| **Papel padrão** | Papel atribuído ao usuário criado pelo JIT ou SCIM |
| **Revalidar SSO a cada** | Intervalo entre novas validações no IdP: 30 minutos, 1, 4, 8 ou 24 horas |
| **Acesso local de emergência do Super Admin** | Permite que o Super Admin continue entrando com a senha local |

### Papel padrão e permissões

- Um usuário novo recebe o **papel padrão** selecionado.
- O IdP não envia nem substitui papéis na versão atual.
- Um administrador pode alterar o papel depois em **Configurações → Usuários**.
- A sincronização seguinte não desfaz essa alteração.
- O papel **Super Admin** não pode ser atribuído por JIT ou SCIM.
- Use o princípio do menor privilégio: prefira um papel como `Tester` ou outro papel customizado restrito e aumente as permissões depois, quando necessário.

### Vínculo do Super Admin

O primeiro teste cria o vínculo entre a conta local do Super Admin e a identidade externa. O e-mail retornado pelo IdP deve ser exatamente o mesmo e-mail cadastrado no QANode, desconsiderando apenas diferenças entre maiúsculas e minúsculas.

Depois do primeiro vínculo, o QANode usa também o identificador estável do provedor (`sub` no OIDC ou `NameID` no SAML). Por isso, não configure um identificador temporário ou que mude a cada login.

### Acesso local de emergência

Se habilitado, somente o Super Admin pode continuar usando a senha local. Essa opção serve para recuperar a administração quando o IdP estiver indisponível ou quando a configuração precisar ser corrigida.

Recomendações:

- mantenha a senha em um cofre corporativo;
- limite quem tem acesso a ela;
- teste periodicamente o procedimento de emergência;
- desabilite a opção se a política da empresa proibir autenticação local.

---

## Provisionamento JIT

No modo **JIT**, o usuário nasce no QANode quando conclui o primeiro login SSO.

### Primeiro acesso

1. O usuário inicia o login no QANode.
2. O IdP autentica a pessoa e devolve um identificador estável, e-mail e nome.
3. Se já existir um usuário com o mesmo e-mail, o QANode vincula a identidade externa a esse cadastro.
4. Se não existir, o QANode cria um usuário ativo, sem senha local e com o papel padrão.
5. Nos próximos acessos, o vínculo usa o identificador imutável do provedor.

O nome e o e-mail podem ser atualizados pelos dados do SSO. O papel permanece sob controle do QANode.

### O que acontece quando o usuário é bloqueado no IdP

O JIT **não envia uma desativação para o banco do QANode**. O registro pode continuar aparecendo como ativo, mas o usuário não conseguirá renovar a autenticação se o IdP bloquear o login.

O tempo máximo para uma sessão já aberta voltar ao IdP depende de **Revalidar SSO a cada**. Para reduzir a janela de acesso sem SCIM, use 30 ou 60 minutos.

Também não use a desativação manual no QANode como fonte de verdade para um usuário JIT que continua autorizado no IdP: um novo login SSO válido pode reativar o cadastro. Remova primeiro o acesso ao aplicativo no IdP.

> Se a desativação do cadastro local precisa ocorrer automaticamente, use SCIM. JIT é uma estratégia de criação no login, não uma sincronização completa do ciclo de vida.

---

## Provisionamento SCIM

No modo **SCIM**, o IdP chama a API SCIM 2.0 do QANode para criar, atualizar, ativar ou desativar usuários.

### Dados fornecidos pelo QANode

| Campo | Uso no provedor |
|-------|-----------------|
| **URL base do SCIM** | URL do endpoint de provisionamento, terminada em `/api/scim/v2` |
| **Token secreto** | Token Bearer usado pelo provedor para autenticar as chamadas |

O token é mostrado somente no momento da geração. Copie-o diretamente para o IdP e armazene-o com segurança. Ao rotacionar o token, o valor anterior deixa de funcionar.

### Recursos suportados

O QANode gerencia usuários SCIM e oferece:

- criação e consulta de usuários;
- atualização completa ou parcial;
- ativação e desativação pelo campo `active`;
- exclusão SCIM tratada como **desativação**, sem apagar fisicamente o usuário;
- filtros necessários para localizar usuários por identidade.

Na versão atual, o QANode não sincroniza grupos, papéis, senhas, fotos ou permissões pelo SCIM. Desative o mapeamento de grupos no provedor.

### Mapeamento recomendado

Mantenha apenas os atributos necessários:

| Atributo SCIM no QANode | Obrigatório | Origem recomendada no IdP | Observação |
|-------------------------|-------------|----------------------------|------------|
| `userName` | Sim | E-mail corporativo real | Deve ser um e-mail válido e único; use como atributo de correspondência, precedência 1 |
| `active` | Recomendado | Estado ativo do usuário | Controla ativação e desativação |
| `displayName` | Recomendado | Nome de exibição | Nome mostrado no QANode |
| `name.givenName` | Não | Nome | Opcional |
| `name.familyName` | Não | Sobrenome | Opcional |
| `emails[type eq "work"].value` | Não | Mesmo e-mail de `userName` | Ajuda provedores que enviam uma coleção de e-mails |
| `externalId` | Recomendado | ID imutável do objeto no IdP | Não use um apelido que possa ser alterado |

O QANode exige que `userName` ou o e-mail principal recebido seja um e-mail válido. Um UPN técnico como `GUID@tenant.onmicrosoft.com` não deve ser usado quando não representa o e-mail real de login.

### Desativação e reativação

- Quando o IdP envia `active: false`, o QANode desativa o usuário e invalida o acesso.
- Quando envia `active: true`, o usuário volta a ficar ativo, respeitando o limite de usuários da licença.
- A operação `DELETE` do SCIM também desativa o usuário; ela não exclui definitivamente seus dados.
- Usuários controlados pelo SCIM devem ser ativados ou desativados no IdP, não manualmente no QANode.
- O Super Admin nunca é desativado ou gerenciado pelo SCIM.

### Janelas de sincronização

O tempo de sincronização é definido pelo provedor. Alguns provedores trabalham em ciclos e a alteração pode levar alguns minutos. Use a função de provisionamento sob demanda do IdP durante a implantação e aguarde o próximo ciclo para validar a rotina normal.

---

## Configurando OIDC

OIDC é a opção recomendada quando o provedor suporta o protocolo. O QANode usa o fluxo **Authorization Code com PKCE** e solicita os escopos `openid`, `profile` e `email`.

### Campos do QANode

| Campo no QANode | Onde encontrar no IdP |
|------------------|------------------------|
| **URL do emissor** | Valor `issuer` do documento `/.well-known/openid-configuration` |
| **Client ID** | ID público do aplicativo OIDC |
| **Client secret** | Valor do segredo do aplicativo; não use o ID ou o nome do segredo |
| **Autenticação do endpoint de token** | `client_secret_post` ou `client_secret_basic`, conforme suportado pelo provedor |
| **URI de redirecionamento** | Copie do QANode para a lista de callbacks/redirect URIs do IdP |

> A URL do emissor deve ser exatamente o valor `issuer` publicado pelo provedor, sem acrescentar ou remover barras, caminhos ou versões manualmente.

O IdP deve retornar:

- um `sub` estável e não vazio;
- um e-mail no claim `email` ou, como alternativa, em `preferred_username`;
- um nome em `name`, quando disponível;
- `email_verified` diferente de `false`, quando o claim for enviado.

### Microsoft Entra ID e External ID

1. Abra **Registros de aplicativo** e selecione o aplicativo do QANode.
2. Em **Autenticação**, adicione a **URI de redirecionamento** do QANode na plataforma **Web**.
3. Em **Certificados e segredos**, crie um segredo e copie o campo **Valor**. O valor só é mostrado uma vez; o **ID do segredo** não funciona como Client Secret.
4. Copie o **ID do aplicativo (cliente)** para o campo Client ID.
5. Abra **Pontos de extremidade** e acesse o **Documento de metadados do OpenID Connect**.
6. No JSON, copie o valor exato de `issuer` para **URL do emissor**.

Em tenants Microsoft Entra de força de trabalho, o emissor normalmente segue este formato:

```text
https://login.microsoftonline.com/{tenant-id}/v2.0
```

No Microsoft Entra External ID, ele normalmente usa o domínio `ciamlogin.com`:

```text
https://{tenant}.ciamlogin.com/{tenant-id}/v2.0
```

Use sempre o valor publicado pelo documento do seu tenant. Comece com `client_secret_post`; altere para `client_secret_basic` somente se o documento do provedor indicar suporte e a troca for necessária.

### Google

1. No Google Cloud Console, configure a tela de consentimento OAuth.
2. Crie um **OAuth Client ID** do tipo **Web application**.
3. Adicione a URI do QANode em **Authorized redirect URIs**.
4. Copie o Client ID e o Client Secret.
5. Use o emissor:

```text
https://accounts.google.com
```

O Google aceita `client_secret_post` e `client_secret_basic`. A URI de redirecionamento deve coincidir exatamente, inclusive protocolo, domínio, porta, caminho e barra final.

### Okta

1. Crie uma integração **OIDC — Web Application**.
2. Em **Sign-in redirect URIs**, adicione a URI do QANode.
3. Atribua os usuários ou grupos à aplicação.
4. Copie Client ID e Client Secret.
5. Use o **Issuer URI** do Authorization Server escolhido. Para o servidor padrão, geralmente é:

```text
https://{seu-dominio-okta}/oauth2/default
```

### Keycloak

1. Crie um client do tipo **OpenID Connect**.
2. Habilite **Standard flow** e autenticação do client.
3. Adicione a URI do QANode em **Valid redirect URIs**.
4. Copie o Client ID e o segredo da aba de credenciais.
5. Use como emissor:

```text
https://{host-keycloak}/realms/{realm}
```

### Auth0

1. Crie uma aplicação **Regular Web Application**.
2. Adicione a URI do QANode em **Allowed Callback URLs**.
3. Copie Client ID e Client Secret.
4. Use o valor `issuer` do discovery do domínio do tenant ou do domínio customizado:

```text
https://{seu-dominio-auth0}/
```

### Zoho

Crie um cliente OIDC para aplicação web, registre a URI de redirecionamento e copie Client ID e Client Secret. Use o valor `issuer` do documento de discovery do **data center da sua organização**. Não suponha que todas as contas usam o domínio `.com`; organizações europeias, indianas e de outras regiões podem publicar outro domínio.

---

## Configurando SAML 2.0

No SAML, o QANode é o **Service Provider (SP)** e a plataforma corporativa é o **Identity Provider (IdP)**.

### Dados do QANode para cadastrar no IdP

| Campo no QANode | Nome comum no provedor |
|------------------|------------------------|
| **URL de login** | Sign-on URL, Start URL ou Login URL |
| **Emissor / Entity ID** | Identifier, Audience URI, SP Entity ID ou Client ID |
| **URL do serviço de consumidor de asserção (ACS)** | Reply URL, ACS URL, Single sign-on URL ou Recipient URL |
| **URL de metadados** | SP Metadata URL, quando o provedor aceita importação de metadata |
| **URL de logout** | Retorno local ao login; não é um endpoint SAML SLO |

> Na versão atual, o login SAML deve começar no QANode. A abertura direta pelo catálogo de aplicativos do IdP (IdP-initiated) e o botão de teste do próprio provedor podem falhar porque não possuem o estado de login criado pelo QANode.

### Dados do IdP para cadastrar no QANode

| Campo no QANode | Nome comum no provedor |
|------------------|------------------------|
| **URL de SSO do IdP** | Login URL, SSO URL ou SAML 2.0 Endpoint |
| **Emissor do IdP** | IdP Entity ID, Issuer ou Identifier |
| **Certificado de assinatura X.509** | Certificate, Signing Certificate ou Certificate (Base64/PEM) |
| **Atributo de e-mail** | Nome exato do atributo que contém o e-mail; o padrão é `email` |
| **Atributo de nome** | Nome exato do atributo que contém o nome; o padrão é `displayName` |

Cole o certificado incluindo `-----BEGIN CERTIFICATE-----` e `-----END CERTIFICATE-----` quando o provedor entregar o arquivo em PEM.

### Requisitos de compatibilidade SAML

Configure o provedor com estas regras:

| Configuração | Valor esperado |
|--------------|----------------|
| Binding do ACS | HTTP POST |
| Asserção SAML | **Assinada** |
| Resposta SAML | Pode estar assinada ou não |
| Criptografia da asserção | Desabilitada |
| AuthnRequest do QANode | Não assinada; o IdP não deve exigir assinatura |
| NameID | Estável, não vazio; formato persistente ou e-mail |
| E-mail | Enviado no atributo configurado ou em um claim de e-mail conhecido |

O QANode procura o e-mail no atributo configurado e, como alternativas, em `email`, `mail`, no OID LDAP de e-mail ou no NameID quando ele tiver formato de e-mail. Mesmo assim, configure explicitamente o atributo para tornar a integração previsível.

### Microsoft Entra ID — SAML

Em **Aplicativos empresariais → Single sign-on → SAML**:

| Campo no Microsoft Entra | Valor do QANode |
|--------------------------|-----------------|
| **Identifier (Entity ID)** | Emissor / Entity ID |
| **Reply URL (ACS)** | URL ACS |
| **Sign-on URL** | URL de login |
| **Logout URL** | Deixe em branco |

Configure os atributos:

| Claim | Origem recomendada |
|-------|--------------------|
| `email` | `user.mail` ou outro atributo que contenha o e-mail real |
| `displayName` | `user.displayname` |

Depois copie para o QANode:

- **Login URL** → URL de SSO do IdP;
- **Microsoft Entra Identifier** → Emissor do IdP;
- certificado de assinatura → Certificado X.509.

Selecione uma opção que assine a **asserção**. Não habilite criptografia e não exija AuthnRequest assinada.

### Google Workspace — SAML

Em **Apps → Web and mobile apps → Add app → Add custom SAML app**:

1. Copie o **SSO URL**, **Entity ID** e o certificado do Google para os campos do IdP no QANode.
2. Configure **ACS URL** com a URL ACS do QANode.
3. Configure **Entity ID** com o Emissor / Entity ID do QANode.
4. Use o e-mail principal como NameID, no formato `EMAIL`.
5. A opção **Signed response** pode ficar desmarcada: o Google continuará assinando a asserção, que é o requisito do QANode.
6. Mapeie o e-mail principal para `email` e o nome de exibição para `displayName`, quando disponível.
7. Habilite o serviço para as unidades organizacionais ou grupos desejados.

Inicie o teste pelo botão do QANode, não pelo botão **Test SAML login** do Google.

### Okta — SAML

| Campo no Okta | Valor |
|----------------|-------|
| **Single sign-on URL** | URL ACS do QANode |
| **Use this for Recipient URL and Destination URL** | Habilitado |
| **Audience URI (SP Entity ID)** | Emissor / Entity ID do QANode |
| **Default RelayState** | Em branco |
| **Name ID format** | EmailAddress |
| **Application username** | Email |
| **Assertion Signature** | Signed |
| **Assertion Encryption** | Unencrypted |
| **Signed Requests** | Desabilitado |

Adicione `email = user.email` e `displayName = user.displayName`. Depois copie a Identity Provider Single Sign-On URL, o Issuer e o certificado X.509 para o QANode.

### OneLogin — SAML

Configure:

- **ACS (Consumer) URL** e **Recipient** com a URL ACS;
- **Audience** com o Entity ID do QANode;
- RelayState em branco;
- NameID com e-mail;
- assinatura da asserção habilitada e criptografia desabilitada.

Copie **SAML 2.0 Endpoint (HTTP)**, **Issuer URL** e o certificado X.509 para o QANode.

### Zoho — SAML

Crie uma aplicação SAML customizada e use:

- **ACS URL** = URL ACS do QANode;
- **Entity ID / Audience** = Emissor do QANode;
- **Start URL** = URL de login do QANode, quando o campo estiver disponível;
- **NameID** = e-mail corporativo;
- atributos `email` e `displayName`.

Copie para o QANode a URL de login SAML, o emissor do Zoho e o certificado fornecido pela plataforma.

### JumpCloud — SAML

Configure SP Entity ID, ACS URL e Login URL com os valores do QANode. Use o e-mail como NameID e selecione uma opção que assine a **assertion** ou a response e a assertion. O padrão que assina somente a response não é suficiente. Deixe a criptografia desligada e não exija requisições assinadas.

### Keycloak — SAML

Crie um client SAML com:

- **Client ID** = Entity ID do QANode;
- **Master SAML Processing URL** e **Valid Redirect URIs** = URL ACS;
- **Client signature required** = desabilitado;
- assinatura de assertions = habilitada;
- criptografia de assertions = desabilitada;
- mappers para `email` e `displayName`.

Use a URL SAML do realm como URL de SSO do IdP, o emissor do realm e o certificado público de assinatura no QANode.

### PingOne, Duo e Auth0 — SAML

Esses provedores usam os mesmos campos do padrão SAML:

- ACS/Recipient/Single Sign-on URL = URL ACS do QANode;
- Audience/Entity ID = Entity ID do QANode;
- NameID = e-mail estável;
- asserção assinada;
- criptografia desabilitada;
- requisição assinada não obrigatória.

Copie a SSO URL, o Issuer/Entity ID do IdP e o certificado X.509 apresentados pelo provedor.

---

## SCIM no Microsoft Entra ID e External ID

O Microsoft Entra configura SCIM em um **Aplicativo empresarial**. Esse aplicativo de provisionamento pode ser separado do registro de aplicativo OIDC.

### Criando a aplicação SCIM

1. Abra **Aplicativos empresariais → Novo aplicativo**.
2. Escolha **Criar seu próprio aplicativo** e selecione a opção para integrar outro aplicativo fora da galeria.
3. Abra **Provisionamento** e escolha o modo automático.
4. Informe a URL base SCIM do QANode em **Tenant URL**.
5. Informe o token gerado pelo QANode em **Secret Token**.
6. Clique em **Test Connection**.
7. Em **Mappings**, mantenha o provisionamento de usuários e desabilite o de grupos.
8. Atribua à aplicação os usuários ou grupos que devem ser sincronizados.
9. Use **Provision on demand** para testar um usuário.
10. Depois da validação, clique em **Start provisioning**.

### Mapeamento mínimo recomendado no Entra

| Destino no QANode | Origem no Entra | Configuração |
|--------------------|-----------------|--------------|
| `userName` | Atributo que contém o e-mail real | Match objects = `Yes`, precedence = `1`, apply = `Always` |
| `active` | `Not([IsSoftDeleted])` | Apply = `Always` |
| `displayName` | `displayName` | Apply = `Always` |
| `emails[type eq "work"].value` | Mesmo atributo de e-mail | Opcional |
| `externalId` | `objectId` ou outro ID imutável exposto | Recomendado |

O atributo computado `IsSoftDeleted` fica verdadeiro quando o usuário é desabilitado, removido da aplicação, excluído ou sai do escopo. Mantenha o mapeamento `active ← Not([IsSoftDeleted])` para que essas ações desativem a conta no QANode.

### Atenção ao e-mail no External ID

Em tenants External ID, é possível que:

- `mail` esteja vazio;
- o `userPrincipalName` seja um identificador técnico como `GUID@tenant.onmicrosoft.com`;
- o e-mail real apareça somente dentro da coleção **Identidades** do usuário.

Nessa situação, `userName ← mail` produzirá valor vazio e o QANode responderá:

```text
A valid userName/email is required
```

Não substitua por um UPN técnico apenas para o teste passar. O valor final enviado a `userName` precisa ser o e-mail real usado pelo usuário.

Procedimento recomendado:

1. Abra **Provision on demand** e selecione o usuário.
2. Inspecione **Modified attributes / Data flow**.
3. Confirme que o valor de destino de `userName` é um e-mail real.
4. Se `mail` estiver vazio, copie ou sincronize o e-mail de login para um atributo de origem que o mecanismo de provisionamento exponha, como `mail` ou uma extensão apropriada.
5. Mapeie esse atributo para `userName` e, opcionalmente, para `emails[type eq "work"].value`.
6. Execute novamente o provisionamento sob demanda antes de iniciar o ciclo automático.

Se a coleção **Identidades** não aparecer como atributo selecionável no mapeamento, ela não pode ser usada diretamente naquela configuração. Nesse caso, o e-mail precisa ser materializado em outra propriedade provisionável ou sincronizado por um processo administrativo do tenant.

### Relação entre o aplicativo OIDC e o aplicativo SCIM

Os dois aplicativos não são vinculados automaticamente pelo portal. Eles trabalham juntos porque usam:

- o mesmo tenant;
- os mesmos usuários ou grupos atribuídos;
- o mesmo e-mail normalizado;
- o QANode como destino comum.

Por isso, atribua os mesmos usuários ou grupos tanto ao aplicativo de login OIDC quanto ao aplicativo empresarial SCIM. O SCIM cria o usuário e o primeiro login OIDC vincula a identidade externa pelo e-mail.

---

## Salvando, Testando e Ativando

O checkbox de SSO não ativa a integração imediatamente. O fluxo seguro é:

1. Preencha os dados.
2. Clique em **Salvar rascunho**.
3. Clique em **Testar configuração**.
4. Entre no IdP com a mesma conta do Super Admin atual.
5. Volte ao QANode e confirme o status **Verificado**.
6. Clique em **Ativar SSO**.

O QANode só permite ativar a mesma configuração que foi testada pelo mesmo Super Admin. Se URL, emissor, client, segredo, certificado ou outro dado do provedor mudar, a verificação é invalidada e um novo teste é obrigatório.

### Efeitos da ativação

Ao ativar o SSO:

- o MFA local é desativado globalmente;
- segredos de MFA locais são removidos;
- a ativação de MFA fica bloqueada enquanto o SSO estiver ligado;
- convites pendentes são invalidados;
- convite, reenvio de convite, aceite e importação de usuários ficam bloqueados;
- sessões locais existentes são invalidadas;
- usuários passam a entrar por OIDC ou SAML;
- somente o Super Admin pode manter login local, se o acesso de emergência estiver habilitado.

O MFA deve ser exigido no próprio IdP por política de acesso condicional ou mecanismo equivalente.

### Alterando uma configuração ativa

Uma alteração nos dados do provedor desativa o SSO automaticamente para evitar que uma configuração não verificada bloqueie toda a organização. Depois da alteração:

1. salve o rascunho;
2. teste novamente;
3. reative o SSO.

Alterar papel padrão ou intervalo de revalidação não modifica papéis já atribuídos.

---

## Operação e Manutenção

### Entrada dos usuários

O usuário deve iniciar em **Entrar com SSO** na página de login do QANode. Para SAML, não use o tile do aplicativo no portal do IdP como fluxo principal.

### Rotação do certificado SAML

O QANode mantém um certificado de assinatura do IdP por configuração. Antes do certificado atual expirar:

1. gere ou ative o novo certificado no IdP;
2. copie o novo certificado para o QANode;
3. salve o rascunho — o SSO será desativado por segurança;
4. teste a integração;
5. ative novamente;
6. revogue o certificado antigo no IdP depois da validação.

Planeje uma janela curta de manutenção, pois não há sobreposição automática de dois certificados na versão atual.

### Rotação do token SCIM

1. Gere um novo token no QANode.
2. Atualize imediatamente o Secret Token no IdP.
3. Teste a conexão.
4. Execute um provisionamento sob demanda.

O token anterior deixa de ser aceito após a rotação.

### Desativando o SSO

Ao desativar o SSO, a autenticação local volta a ser permitida. Usuários criados somente por JIT ou SCIM não possuem senha local e precisarão passar pelo processo de recuperação/definição de senha antes de entrar localmente.

O token SCIM não é removido automaticamente ao desligar o SSO. Revogue-o separadamente se o provisionamento também precisar parar.

---

## Solução de Problemas

| Sintoma | Causa provável | Como corrigir |
|---------|----------------|---------------|
| `redirect_uri_mismatch` ou `AADSTS50011` | URI cadastrada no IdP é diferente da enviada pelo QANode | Copie novamente a URI do QANode e cadastre-a como callback **Web**, sem alterar nenhum caractere |
| Teste OIDC falha antes da tela de login | Issuer incorreto ou discovery inacessível | Abra `issuer/.well-known/openid-configuration` e confirme que o `issuer` retornado é exatamente o configurado |
| `invalid_client` | Client ID ou segredo incorreto | Use o **valor** do segredo, confirme validade e método `client_secret_post/basic` |
| E-mail do Super Admin não corresponde | IdP devolveu outro claim ou alias | Ajuste o usuário/claim para retornar o mesmo e-mail cadastrado no QANode |
| SAML falha no botão de teste do provedor | Fluxo iniciado pelo IdP, sem estado criado pelo QANode | Inicie em **Testar configuração** no QANode |
| SAML informa assinatura inválida ou ausente | Somente a response foi assinada | Configure o IdP para assinar a assertion |
| SAML não processa a resposta | Assertion criptografada ou AuthnRequest assinada obrigatória | Desabilite criptografia e a exigência de assinatura da requisição |
| SCIM retorna `A valid userName/email is required` | `userName` vazio ou contendo UPN técnico | Mapeie um atributo de origem que entregue o e-mail real e valide o valor de destino no provisionamento sob demanda |
| Entra mostra `Active in the source system = False` | Usuário desabilitado, excluído ou fora do escopo | Verifique atribuição à aplicação, estado da conta, filtros e `IsSoftDeleted` |
| Entra mostra `UnprocessableEntry` por atributo de correspondência | Origem do atributo de match está vazia | Corrija `userName`, mantenha match precedence 1 e execute novamente sob demanda |
| Usuário foi removido do IdP, mas aparece ativo no QANode com JIT | JIT não sincroniza desativação | Use SCIM ou aguarde a revalidação da sessão; para controle completo, migre o provisionamento para SCIM |
| Usuário SCIM não foi criado | Não está atribuído, está fora do escopo ou excedeu a licença | Confirme atribuição, filtros, status, logs de provisioning e assentos disponíveis |
| Papel não mudou após alteração no IdP | Papéis não são sincronizados | Altere o papel no QANode |
| O SSO desativou depois de salvar | Configuração crítica foi alterada | Teste novamente e reative |

---

## Checklist de Homologação

Antes de liberar para todos os usuários, valide:

- [ ] Super Admin consegue testar e entrar pelo SSO.
- [ ] Acesso local de emergência foi testado ou conscientemente desabilitado.
- [ ] Um usuário comum novo recebe o papel padrão esperado.
- [ ] Um usuário já existente com o mesmo e-mail é vinculado sem duplicação.
- [ ] Alterar o papel no QANode não é desfeito pelo IdP.
- [ ] Usuário sem atribuição ao aplicativo não consegue autenticar.
- [ ] No SCIM, criar usuário gera o cadastro correto no QANode.
- [ ] No SCIM, `active: false` desativa o usuário.
- [ ] No SCIM, reativar no IdP reativa o usuário.
- [ ] O mapeamento envia um e-mail real em `userName`.
- [ ] A política de MFA está aplicada no IdP.
- [ ] A revalidação de sessão atende à política de desligamento da empresa.
- [ ] Responsáveis e prazo de rotação do segredo, token SCIM e certificado SAML estão definidos.

---

## Referências dos Provedores

- [Microsoft Entra — protocolo OIDC](https://learn.microsoft.com/entra/identity-platform/v2-protocols-oidc)
- [Microsoft Entra — mapeamentos de atributos SCIM](https://learn.microsoft.com/entra/identity/app-provisioning/customize-application-attributes)
- [Microsoft Entra — provisionamento SCIM](https://learn.microsoft.com/entra/identity/app-provisioning/use-scim-to-provision-users-and-groups)
- [Google — OpenID Connect](https://developers.google.com/identity/openid-connect/openid-connect)
- [Google Workspace — aplicativo SAML customizado](https://support.google.com/a/answer/6087519)
- [Okta — referência de campos SAML](https://help.okta.com/en-us/content/topics/apps/aiw-saml-reference.htm)
- [Keycloak — guia de administração](https://www.keycloak.org/docs/latest/server_admin/)
- [JumpCloud — campos do conector SAML](https://jumpcloud.com/support/sso-application-connector-fields)
- [Duo — SSO SAML genérico](https://duo.com/docs/sso-generic)
