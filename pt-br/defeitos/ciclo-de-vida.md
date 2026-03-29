# Ciclo de Vida do Defeito

Esta página descreve o caminho completo de um defeito — desde a abertura até o encerramento — incluindo como a atribuição, as filas, o claim e a tramitação funcionam na prática.

---

## 1. Abrindo um defeito

### A partir de uma execução com falha

Esta é a forma mais comum de abrir um defeito. Quando uma execução termina com falha:

1. Acesse o detalhe da execução
2. Clique em **Abrir Defeito**
3. Preencha os campos exigidos pelo workflow (definidos no status inicial)
4. Clique em **Salvar**

O defeito é criado com vínculo direto à execução, ao fluxo e, quando aplicável, ao step que originou a falha. Esse vínculo é preservado mesmo que o fluxo seja editado depois — o defeito guarda uma foto do estado original do cenário.

> **Permissão necessária:** `bug.create`

### Manualmente

Para abrir um defeito sem vínculo com uma execução:

1. Acesse **Defeitos** no menu lateral
2. Clique em **Novo Defeito**
3. Preencha os campos obrigatórios
4. Clique em **Salvar**

---

## 2. Identificação do defeito

Cada defeito recebe automaticamente:

- **Bug Key** — código único de referência no formato `BUG-N` (ex: `BUG-42`), útil para mencionar em comentários, e-mails ou sistemas externos
- **Data de criação** e **reporter** — quem abriu e quando

---

## 3. Atribuição — Filas e Responsáveis

Depois de aberto, um defeito precisa ser direcionado a alguém para ser trabalhado. Isso pode acontecer de duas formas:

### Fila

Uma fila representa uma equipe. Quando um defeito está em fila:
- Qualquer membro daquela fila com `bug.assign` pode **dar claim** e assumir o defeito
- O defeito fica visível para toda a equipe da fila

Filas podem ser aplicadas automaticamente quando o workflow define uma **fila padrão** para um status — ao tramitar para aquele status, o defeito é automaticamente direcionado à fila configurada.
Filas podem ser aplicadas automaticamente quando o workflow define uma **fila padrão** para um status — na ausência de uma escolha manual de roteamento, o defeito é direcionado à fila configurada.

### Responsável direto

Quando um defeito é atribuído diretamente a um usuário:
- Apenas esse usuário pode tramitar no fluxo normal
- Outros usuários, inclusive da mesma fila, não conseguem dar claim no fluxo normal enquanto houver responsável direto

### Sem atribuição

Um defeito sem fila e sem responsável pode ser claimado por qualquer usuário com `bug.assign`.

---

## 4. Claim — Assumindo o defeito

**Claim** é o ato de colocar o defeito no seu nome para trabalhar nele. A tramitação normal exige que o defeito esteja claimado no usuário que vai tramitar.

Para dar claim:

1. Abra o detalhe do defeito
2. Clique em **Assumir**

### Quando posso dar claim?

| Situação do defeito | Quem pode dar claim |
|--------------------|---------------------|
| Em fila, sem responsável | Qualquer membro daquela fila com `bug.assign` |
| Sem fila e sem responsável | Qualquer usuário com `bug.assign` |
| Com responsável direto | Ninguém no fluxo normal; apenas quem tem `bug.transition_any` pode atuar por bypass administrativo |
| Em status final | Ninguém — status final bloqueia claim |

### O que o claim faz

- Coloca seu usuário como responsável direto do defeito
- Mantém a fila associada (o defeito continua vinculado à equipe)

### O que o claim não faz

O claim não remove a fila associada ao defeito. Porém, se existir responsável direto, essa posse passa a prevalecer no fluxo normal. Na prática, outros membros da mesma fila não conseguem mais claimar ou tramitar enquanto o defeito estiver no nome de alguém.

---

## 5. Tramitação — Mudando o status

Tramitar significa mover o defeito de um status para outro seguindo as transições definidas no workflow.

### Como tramitar

1. Abra o detalhe do defeito
2. Os botões de tramitação disponíveis aparecem baseados nas transições que partem do status atual
3. Clique na transição desejada (ex: "Iniciar Investigação", "Corrigir", "Fechar")
4. Preencha os campos obrigatórios da transição (se houver)
5. Adicione uma nota opcional
6. Confirme

### Regras de tramitação

**Para tramitar no fluxo normal (`bug.transition`):**
- O defeito precisa estar claimado no seu usuário
- Se o defeito estiver apenas em fila (sem responsável direto), você precisa primeiro dar claim
- Se o defeito estiver com outro responsável, você não consegue tramitar

**Para tramitar com bypass administrativo (`bug.transition_any`):**
- Você pode tramitar para qualquer status, independente de quem é o responsável
- Não precisa de claim prévio
- Útil para gestores e administradores do processo

### O que acontece após a tramitação

- O status do defeito é atualizado
- Se a transição não informar manualmente um novo responsável ou fila, e o status de destino tiver uma **fila padrão**, o defeito é direcionado para ela
- Se o status de destino for **final**, a data de fechamento é registrada
- Se o defeito voltar de um status final para um status ativo, a data de fechamento é removida
- A transição gera um evento no **histórico** do defeito com a nota informada

---

## 6. Situações práticas

### Defeito parado em fila de QA, sem responsável

1. Um usuário de QA com `bug.assign` dá claim → defeito fica no nome dele
2. Ele investiga e clica em "Corrigir" → defeito vai para o status seguinte
3. Se o próximo status tiver fila padrão de Dev, o defeito é automaticamente direcionado à fila de Dev

### Defeito com responsável direto

1. Apenas esse responsável consegue tramitar no fluxo normal
2. Outro usuário com `bug.transition` não consegue dar claim nem tramitar enquanto já houver responsável
3. Um gerente com `bug.transition_any` pode avançar o defeito sem restrição

### Defeito encerrado sendo reaberto

1. Um usuário com `bug.transition_any` tramita de volta para um status ativo (ex: "Reaberto")
2. A data de fechamento é automaticamente removida
3. O histórico registra a reabertura com quem fez e quando

---

## 7. Lista de defeitos

A tela de listagem de defeitos exibe todos os defeitos da instância com filtros por:

- **Status** — filtre por um ou mais status do workflow
- **Severidade** — crítico, alto, médio, baixo
- **Prioridade** — urgência do defeito
- **Fila** — equipe responsável
- **Responsável** — usuário atribuído

Cada linha da lista mostra o Bug Key, título, status, severidade, prioridade, responsável e data de criação. Clique em qualquer defeito para abrir o detalhe.

---

## 8. Detalhe do defeito

A tela de detalhe do defeito reúne tudo em um único lugar:

| Seção | O que mostra |
|-------|-------------|
| **Cabeçalho** | Bug Key, título, status atual, reporter e data |
| **Informações** | Severidade, prioridade, responsável, fila, projeto e execução de origem |
| **Campos customizados** | Campos adicionais definidos pela empresa |
| **Transições** | Botões para tramitar para o próximo status |
| **Sandbox** | Botão para abrir investigação em ambiente isolado |
| **Comentários** | Discussão sobre o defeito |
| **Anexos** | Arquivos enviados ao defeito |
| **Histórico** | Trilha completa de todas as alterações e transições |
