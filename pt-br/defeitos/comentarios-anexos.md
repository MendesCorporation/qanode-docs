# Comentários, Anexos e Histórico

> **Permissão necessária:** `bug.view`

Toda colaboração e toda movimentação de um defeito ficam registradas no seu detalhe. Esta página explica como usar comentários, anexos e o histórico para acompanhar e documentar o ciclo de vida de um defeito.

---

## Comentários

Os comentários são o espaço de discussão do defeito. Use para:

- Registrar descobertas durante a investigação
- Comunicar bloqueios para a equipe
- Documentar hipóteses testadas
- Coordenar handoffs entre equipes

### Como comentar

1. Acesse o detalhe do defeito
2. Role até a seção **Comentários**
3. Digite sua mensagem no campo de texto
4. Clique em **Comentar**

O comentário é salvo com seu nome, foto de perfil e data/hora. Além de aparecer na seção de comentários, ele também gera um evento no **histórico** do defeito.

---

## Anexos

Anexos permitem incluir arquivos relevantes diretamente no defeito — capturas de tela, logs, vídeos de reprodução, relatórios de erro, entre outros.

### Como anexar um arquivo

1. Acesse o detalhe do defeito
2. Role até a seção **Anexos**
3. Clique em **Adicionar Anexo** ou arraste o arquivo para a área indicada
4. O arquivo é enviado e fica disponível para todos com acesso ao defeito

### Excluindo anexos

A exclusão de anexos segue uma regra de autoria:

| Quem pode excluir | Condição |
|-------------------|----------|
| O próprio usuário que enviou | Pode excluir qualquer arquivo que ele mesmo enviou |
| Admin / Super Admin com `bug.delete_attachment_any` | Pode excluir qualquer anexo de qualquer pessoa |
| Outros usuários | Não podem excluir arquivos enviados por terceiros |

Para excluir:
1. Localize o anexo na seção **Anexos**
2. Clique no ícone de exclusão ao lado do arquivo
3. Confirme a ação

---

## Histórico

O histórico é a trilha auditável completa do defeito. Cada evento registrado mostra quem fez, o que fez e quando.

### O que gera um evento no histórico

| Ação | Evento gerado |
|------|--------------|
| Defeito aberto | "Defeito criado por [usuário]" |
| Tramitação de status | "Status alterado de [X] para [Y] por [usuário]" + nota da transição |
| Claim | "Assumido por [usuário]" |
| Atribuição de responsável | "Responsável alterado para [usuário]" |
| Mudança de fila | "Fila alterada para [fila]" |
| Comentário adicionado | "Comentário de [usuário]" |
| Sandbox criado | "Sandbox de investigação criado" |
| Sandbox descartado | "Sandbox de investigação descartado" |
| Anexo adicionado | Gera evento no histórico com o nome do arquivo; o arquivo também aparece na seção **Anexos** |
| Anexo removido | Gera evento no histórico com o nome do arquivo removido; a remoção também afeta a seção **Anexos** |

### Como acessar o histórico

O histórico fica na seção **Histórico** no detalhe do defeito, em ordem cronológica. Você pode ver toda a linha do tempo do defeito desde a criação até o momento atual.

---

## Nota de transição

Ao tramitar um defeito (mudar o status), você pode adicionar uma **nota de transição**. Essa nota:

- Aparece no histórico junto com o evento de mudança de status
- É opcional nas transições
- Ajuda a documentar o motivo da mudança — por exemplo, ao rejeitar um defeito, explique por que ele não foi reproduzível

---

## Boas práticas

- **Comente antes de tramitar** — se a tramitação vai mudar a posse do defeito para outra equipe, registre o contexto da investigação num comentário antes de passar adiante
- **Use a nota de transição para decisões** — ao fechar, corrigir ou rejeitar, a nota de transição é o lugar certo para registrar o que foi feito ou descoberto
- **Anexe evidências de reprodução** — capturas de tela e logs no sandbox são temporários; se forem relevantes para o registro definitivo, baixe e anexe diretamente no defeito
- **O histórico é imutável** — comentários e eventos de histórico não podem ser editados ou excluídos, garantindo rastreabilidade completa
