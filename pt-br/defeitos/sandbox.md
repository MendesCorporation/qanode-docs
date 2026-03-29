# Sandbox de Investigação

> **Permissão necessária:** `bug.run_sandbox`

O sandbox é um ambiente isolado de investigação vinculado a um defeito. Ele permite que você execute o fluxo original que causou a falha, observe o comportamento e faça ajustes — sem afetar os dados oficiais, sem contaminar os relatórios e sem interferir com as execuções reais do projeto.

---

## O que é o sandbox

Quando um defeito é aberto a partir de uma execução com falha, o QANode preserva uma **foto do fluxo original** no momento em que o defeito foi criado. O sandbox parte sempre dessa foto, não do fluxo atual.

Isso significa que:

- Mesmo que alguém edite ou exclua o fluxo original depois, o sandbox continua baseado no estado exato que gerou o problema
- A investigação parte sempre do cenário que reproduz a falha, não de uma versão mais nova que pode ter mudado

---

## Abrindo o sandbox

1. Acesse o detalhe do defeito
2. Clique em **Abrir Sandbox**
3. O QANode cria uma cópia do fluxo original marcada como sandbox
4. Você é redirecionado para o editor com esse fluxo

Se já existir um sandbox ativo para o defeito, o sistema reabre o existente em vez de criar um novo.

---

## O que você pode fazer no sandbox

Dentro do sandbox você tem acesso ao editor de fluxos completo e a atalhos diretos para o defeito:

- **Executar** o fluxo para reproduzir a falha
- **Inspecionar** os nós e ver os resultados passo a passo
- **Modificar** o fluxo para testar hipóteses de correção
- **Re-executar** quantas vezes precisar
- **Adicionar comentário no defeito** sem sair do sandbox (botão 💬 na barra de ferramentas)
- **Adicionar anexo no defeito** sem sair do sandbox (botão 📎 na barra de ferramentas)

### Comentário direto no defeito

Clique no botão de **comentário** (ícone de balão) na barra de ferramentas. Um modal abre para você digitar e salvar o comentário diretamente no defeito vinculado ao sandbox — sem precisar voltar à tela do defeito.

Use para registrar descobertas enquanto ainda está investigando: o que reproduziu, o que testou, o que encontrou.

### Anexo direto no defeito

Clique no botão de **clipe** na barra de ferramentas e selecione o arquivo. O arquivo é enviado diretamente ao defeito vinculado. Útil para anexar capturas de tela, logs ou evidências coletadas durante a investigação, sem sair do sandbox.

Todas as execuções feitas dentro do sandbox:
- Ficam marcadas como `sandbox` internamente
- **Não aparecem nos relatórios e dashboards oficiais**
- **Não contam nos KPIs do projeto**
- **Não afetam o fluxo original**

---

## Quem pode abrir o sandbox

O acesso ao sandbox segue as mesmas regras de posse do defeito:

| Situação do defeito | Quem pode abrir sandbox |
|--------------------|------------------------|
| Em fila, sem responsável | Qualquer membro daquela fila com `bug.run_sandbox` |
| Atribuído diretamente a um usuário | Apenas esse usuário no fluxo normal |
| Em fila de outra equipe | Usuários de fora da fila não conseguem no fluxo normal |
| Em status final | Ninguém — sandbox é bloqueado em status final |
| Usuário com `bug.transition_any` | Pode bypassar a posse, mas ainda precisa ter `bug.run_sandbox` |

---

## Descartando o sandbox

Quando a investigação terminar:

1. Acesse o detalhe do defeito
2. Clique em **Descartar Sandbox**
3. Confirme a ação

O descarte:
- Remove o fluxo sandbox
- Remove todas as execuções sandbox vinculadas
- Registra o evento no histórico do defeito

O descarte é definitivo — o sandbox removido não pode ser recuperado. Se precisar investigar novamente, um novo sandbox pode ser criado a qualquer momento (enquanto o defeito não estiver em status final).

---

## Diferença entre sandbox e execução oficial

| | Execução oficial | Execução sandbox |
|-|-----------------|-----------------|
| Aparece nos relatórios | ✅ Sim | ❌ Não |
| Conta nos KPIs do projeto | ✅ Sim | ❌ Não |
| Parte do fluxo atual | ✅ Sim | ❌ Usa foto do original |
| Pode ser modificada livremente | ❌ Não | ✅ Sim |

---

## Boas práticas

- **Não descarte o sandbox antes de documentar** — se você encontrou a causa da falha, adicione um comentário no defeito com as suas conclusões antes de descartar
- **Use o sandbox para reproduzir antes de corrigir** — confirmar que você consegue reproduzir a falha no sandbox é o primeiro passo de uma boa investigação
- **Um sandbox por vez** — o sistema reutiliza o sandbox existente. Se você precisar de um estado limpo, descarte o atual e abra um novo
