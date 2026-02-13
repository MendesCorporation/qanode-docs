# Suítes de Teste

**Suítes** permitem agrupar múltiplos cenários (fluxos) para execução sequencial ou não sequencial. São ideais para regressão, smoke tests e execuções agendadas.

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
| **Modo de Execução** | Não | Sequencial (padrão) ou Não sequencial |

4. Clique em **Criar**

<!-- ![Criar suíte](../../assets/images/criar-suite.png) -->
*Imagem: Formulário de criação de suíte com seleção de cenários e toggle de parar em falha*

---

## Modo de Execução

A suíte oferece dois modos de execução:

### Sequencial (padrão)

Os cenários são executados **um por vez**, na ordem definida. Você pode reordenar arrastando os cenários na lista.

```
Suíte: Regressão Login (Sequencial)
├── 1. Login com credenciais válidas    ⏳ aguardando
├── 2. Login com senha incorreta        ⏳ aguardando
├── 3. Login com email inválido         ⏳ aguardando
├── 4. Recuperação de senha             ⏳ aguardando
└── 5. Logout                           ⏳ aguardando
→ Executa: 1, depois 2, depois 3...
```

Use o modo sequencial quando:
- Os cenários **dependem uns dos outros** (ex: login antes de operações)
- Você precisa de **ordem garantida** de execução
- Quer usar a opção **Parar em Falha** para interromper na primeira falha

### Não sequencial

Os cenários são executados **de forma independente**, sem ordem garantida. Todos os cenários são disparados e executados paralelamente.

```
Suíte: Regressão Login (Não sequencial)
├── 1. Login com credenciais válidas    ▶ executando
├── 2. Login com senha incorreta        ▶ executando
├── 3. Login com email inválido         ▶ executando
├── 4. Recuperação de senha             ▶ executando
└── 5. Logout                           ▶ executando
→ Todos executam ao mesmo tempo
```

Use o modo não sequencial quando:
- Os cenários são **independentes** entre si
- Você quer **reduzir o tempo total** de execução da suíte
- A ordem de execução **não importa**

---

## Parar em Falha

> **Nota:** A opção "Parar em Falha" tem efeito apenas no modo **Sequencial**. No modo Não sequencial, como todos os cenários são executados de forma independente, esta opção não se aplica.

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

- **Ordene estrategicamente** — no modo sequencial, coloque cenários de setup (login, preparação) primeiro
- **Use "Parar em Falha"** quando cenários dependem uns dos outros (modo sequencial)
- **Desative "Parar em Falha"** para suítes de regressão completa
- **Use modo não sequencial** para suítes de regressão com cenários independentes — reduz o tempo total
- **Agende em horários de baixa** — evite conflito com uso manual
- **Monitore com alarmes** — configure notificações para falhas em suítes agendadas - versão Enterprise
