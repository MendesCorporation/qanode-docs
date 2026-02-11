# Suítes de Teste

**Suítes** permitem agrupar múltiplos cenários (fluxos) para execução sequencial. São ideais para regressão, smoke tests e execuções agendadas.

---

## Criando uma Suíte

1. No menu lateral, clique em **Suítes**
2. Clique em **+ Nova Suíte**
3. Preencha:

| Campo | Obrigatório | Descrição |
|-------|-------------|-----------|
| **Nome** | Sim | Nome da suíte |
| **Projeto** | Sim | Projeto ao qual a suíte pertence |
| **Cenários** | Sim | Selecione os cenários e a ordem |
| **Parar em Falha** | Não | Interromper ao primeiro cenário que falhar |

4. Clique em **Criar**

<!-- ![Criar suíte](../../assets/images/criar-suite.png) -->
*Imagem: Formulário de criação de suíte com seleção de cenários e toggle de parar em falha*

---

## Ordem de Execução

Os cenários dentro de uma suíte são executados **sequencialmente**, na ordem definida. Você pode reordenar arrastando os cenários na lista.

```
Suíte: Regressão Login
├── 1. Login com credenciais válidas
├── 2. Login com senha incorreta
├── 3. Login com email inválido
├── 4. Recuperação de senha
└── 5. Logout
```

---

## Parar em Falha

Quando **Parar em Falha** está ativado:
- Se um cenário falhar, os cenários seguintes **não são executados**
- Útil quando cenários dependem uns dos outros (ex: login antes de operações)

Quando desativado:
- Todos os cenários são executados, independentemente de falhas
- Útil para suítes de regressão completa onde você quer ver todas as falhas

---

## Executando uma Suíte

### Execução Manual

1. Na lista de suítes, clique no botão **Executar** (▶️)
2. Todos os cenários serão executados em ordem
3. O resultado aparece na lista de execuções

### Execução Agendada

Suítes podem ser executadas automaticamente em horários definidos.

---

## Agendamento

### Configurando o Agendamento

1. Edite a suíte
2. Na seção **Agendamento**, ative o toggle
3. Informe a **expressão cron**
4. O sistema mostrará os próximos 5 horários de execução

[Agendamento de suíte](../../assets/images/suites.mp4) 
*Imagem: Seção de agendamento com campo cron, toggle de ativação e preview dos próximos horários*

### Expressões Cron

O formato cron possui 5 campos:

```
┌───────────── minuto (0-59)
│ ┌─────────── hora (0-23)
│ │ ┌───────── dia do mês (1-31)
│ │ │ ┌─────── mês (1-12)
│ │ │ │ ┌───── dia da semana (0-7, 0 e 7 = domingo)
│ │ │ │ │
* * * * *
```

### Exemplos Comuns

| Expressão | Significado |
|-----------|-------------|
| `0 8 * * *` | Todos os dias às 8:00 |
| `0 8 * * 1-5` | Segunda a sexta às 8:00 |
| `0 */2 * * *` | A cada 2 horas |
| `30 9 * * 1` | Toda segunda às 9:30 |
| `0 0 * * *` | Todo dia à meia-noite |
| `0 6,12,18 * * *` | Às 6h, 12h e 18h |
| `*/15 * * * *` | A cada 15 minutos |
| `0 9 1 * *` | Primeiro dia do mês às 9:00 |

### Ativar/Desativar

Você pode ativar e desativar o agendamento sem remover a expressão cron. Quando desativado, a expressão é preservada para reativação futura.

---

## Resultados da Suíte

Após a execução, a suíte exibe:

| Informação | Descrição |
|------------|-----------|
| **Status Geral** | Sucesso (todos passaram) ou Falha (algum falhou) |
| **Cenários Executados** | Quantos foram processados |
| **Cenários com Sucesso** | Quantos passaram |
| **Cenários com Falha** | Quantos falharam |
| **Duração Total** | Tempo total de execução |

Cada cenário individual também possui seu resultado detalhado (logs, screenshots, etc.).

---

## Alarmes

Você pode configurar **alarmes** para ser notificado quando suítes falham. Veja a seção de [Administração](../administracao/administracao.md) para configurar alarmes.

---

## Dicas

- **Ordene estrategicamente** — coloque cenários de setup (login, preparação) primeiro
- **Use "Parar em Falha"** quando cenários dependem uns dos outros
- **Desative "Parar em Falha"** para suítes de regressão completa
- **Agende em horários de baixa** — evite conflito com uso manual
- **Monitore com alarmes** — configure notificações para falhas em suítes agendadas - versão Enterprise
