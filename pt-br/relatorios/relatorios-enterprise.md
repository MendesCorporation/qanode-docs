# Relatórios

O QANode gera **relatórios PDF** automaticamente após cada execução e permite gerar relatórios consolidados por período, com exportação e envio por e-mail.

---

## Relatórios de Execução

Após cada execução de cenário ou suíte, um **relatório PDF** é gerado automaticamente com:

- **Resumo**: Status, duração, data de execução
- **Detalhes por Nó**: Status individual, logs, outputs, duração
- **Screenshots**: Capturas de tela (quando configuradas nos nós web)
- **Erros**: Detalhes das falhas encontradas

O relatório fica disponível na lista de execuções e pode ser baixado a qualquer momento.

---

## Relatórios Consolidados

Gere relatórios que abrangem múltiplas execuções em um período:

1. No menu lateral, clique em **Relatórios**
2. Selecione o **período** (data início e fim)
3. (Opcional) Filtre por **projeto**
4. Clique em **Gerar Relatório**

<!-- ![Gerar relatório](../../assets/images/gerar-relatorio.png) -->
*Imagem: Tela de relatórios com seleção de período, filtro de projeto e botão de gerar*

### Métricas do Relatório

| Métrica | Descrição |
|---------|-----------|
| **Taxa de Sucesso** | Percentual de execuções bem-sucedidas |
| **Total de Execuções** | Quantidade total de execuções no período |
| **Execuções com Sucesso** | Quantidade de execuções aprovadas |
| **Execuções com Falha** | Quantidade de execuções reprovadas |
| **Duração Média** | Tempo médio de execução |
| **Top Falhas por Cenário** | Cenários que mais falharam |
| **Top Falhas por Tipo de Nó** | Tipos de nó que mais falharam |

### Gráfico de Execuções por Dia

O relatório inclui um gráfico de barras mostrando a distribuição de execuções com sucesso e falha ao longo dos dias do período selecionado.

### Métricas por Projeto

Quando um projeto específico é selecionado, métricas adicionais são exibidas:

| Métrica | Descrição |
|---------|-----------|
| **Total de Cenários/Suítes** | Quantidade no projeto |
| **Cenários Concluídos** | Cenários que passaram |
| **Cenários Pendentes** | Cenários ainda não executados |
| **Cenários Reprovados** | Cenários que falharam |
| **Progresso** | % do tempo decorrido vs % cenários executados |

---

## Exportação

### Download PDF

1. Gere o relatório
2. Clique em **Exportar PDF**
3. O arquivo será baixado com o nome: `report_{projeto}_{inicio}_{fim}.pdf`

### Envio por E-mail

1. Gere o relatório
2. Clique em **Enviar por E-mail**
3. Selecione os destinatários:
   - **Usuários cadastrados** — selecione da lista
   - **E-mails adicionais** — adicione endereços customizados
4. Clique em **Enviar**

O relatório PDF será enviado como **anexo** do e-mail.

> **Requisito:** O SMTP deve estar configurado em Configurações para envio de e-mails. Veja [Administração](../administracao/administracao.md).

---

## Template de Relatório

O layout do relatório PDF é configurável. Acesse **Configurações** → **Template de Relatório** para customizar:

| Configuração | Descrição |
|-------------|-----------|
| **Imagem de Fundo** | Imagem de fundo das páginas |
| **Logo** | Logo exibido no cabeçalho |
| **Exibir Logo** | Ativar/desativar logo |
| **Incluir Screenshots** | Incluir capturas de tela |
| **Tamanho dos Screenshots** | Dimensão das imagens no relatório |
| **Incluir Logs** | Incluir logs de cada nó |
| **Máximo de Logs** | Limite de linhas de log por nó |
| **Incluir Outputs** | Incluir dados de saída dos nós |
| **Incluir Duração** | Mostrar duração de cada nó |
| **Incluir ID do Nó** | Mostrar identificador técnico |
| **Numeração de Passos** | Numerar os passos sequencialmente |
| **Seção de Resumo** | Incluir resumo no início |
| **Detalhes de Erros** | Incluir detalhes completos dos erros |
| **Metadados** | Data, ID da execução, nome do projeto/fluxo |

<!-- ![Template de relatório](../../assets/images/template-relatorio.png) -->
*Imagem: Tela de configuração do template com preview das opções*

---

## Dicas

- **Configure o template** antes de gerar relatórios — especialmente logo e imagem de fundo
- **Inclua screenshots** para evidências visuais — fundamental para auditorias
- **Use filtro por projeto** para relatórios focados
- **Agende envios** — use suítes agendadas + alarmes para relatórios automáticos
- **Limite os logs** — relatórios com muitos logs ficam extensos; use 10-20 linhas por nó
