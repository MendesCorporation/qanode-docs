# Instalação


---

## Versão Desktop 

A versão desktop é a forma mais fácil de começar. Ela inclui todos os componentes necessários em um único instalador, incluindo um banco de dados PostgreSQL embutido.

### Requisitos

- **Sistema Operacional**: Windows 10/11 (64-bit)
- **Memória RAM**: 4 GB mínimo (8 GB recomendado)
- **Espaço em Disco**: 1 GB para instalação

### Instalação no Windows

1. Baixe o instalador `QANode Setup.exe` no site https://www.qanode.com
2. Execute o instalador e siga as instruções
3. Ao finalizar, o QANode será aberto automaticamente

[Instalador do QANode no Windows](../../assets/images/instalacao-nsis.png) 
*Imagem: Tela do instalador NSIS do QANode no Windows*

Na primeira execução, o QANode irá:
- Inicializar o banco de dados PostgreSQL embutido
- Criar as tabelas necessárias (migrations automáticas)
- Iniciar o servidor da API
- Abrir a interface do aplicativo

> **Nota:** A primeira inicialização pode levar alguns segundos a mais enquanto o banco de dados é preparado. Uma tela de splash será exibida durante esse processo.

### Pasta de Dados

A versão desktop armazena seus dados em:

```
%USERPROFILE%\AppData\Roaming\QANode\
├── pg-data/         # Dados do PostgreSQL embutido
├── storage/         # Artefatos (screenshots, PDFs, etc.)
└── custom nodes/    # Nós customizados locais
```

> **Importante:** Nunca modifique manualmente os arquivos em `pg-data/`. Use a interface do QANode para gerenciar seus dados.

---


## Solução de Problemas

### A versão desktop não inicia

- Verifique se outra instância do QANode não está em execução
- Tente executar como administrador
- Verifique se a porta 5432 não está em uso por outro PostgreSQL

---

## Próximos Passos

- [Início Rápido](inicio-rapido.md) — Crie seu primeiro teste
- [Conceitos Fundamentais](conceitos.md) — Entenda a estrutura da plataforma
